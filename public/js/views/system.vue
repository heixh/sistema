vueSystem = new Vue({
    el: '#vueSystem',
    data: {
        systemTitle: '',
        systemId: null,
        answerHistory: [],
        arrQuestions: [],
        arrElements: [],
        form: {
            question: {
                id: null,
                question: ''
            },
            answer: null
        }
    },
    methods: {
        sendAnswer() {
            if (this.form.answer !== null) {
                const obj = {
                    answer: this.form.answer,
                    question: this.form.question
                }
                this.answerHistory.push(obj);

                fetch(`/questionElement/answer?questionId=${this.form.question.id}&answer=${this.form.answer}&systemId=${this.systemId}`, {
                    method: "get",
                    headers: { "Content-Type": "application/json" }
                })
                .then((res) => res.json())
                .then((data) => {
                    // Incrementing 1 point to the elements that can be a response:
                    for (const i of this.arrElements) {
                        for (const j of data.elements) {
                            if (j.id === i.id) i.points = i.points + 1;
                        }
                    }

                    // If we have a single element:
                    if (data.elements.length === 1) {
                        const result = {
                            element: data.elements[0],
                            allElements: this.arrElements,
                            history: this.answerHistory,
                        }
                        window.localStorage.setItem('result', JSON.stringify(result));
                        window.location.href = `/system/${this.systemId}/result`;
                    } else if (data.elements.length === 0) {
                        const result = {
                            element: {
                                id: 0,
                                element: 'Not found'
                            },
                            allElements: this.arrElements,
                            history: this.answerHistory,
                        }
                        window.localStorage.setItem('result', JSON.stringify(result));
                        window.location.href = `/system/${this.systemId}/result`
                    }

                    // Updating questions:
                    this.arrQuestions = data.questions;
                    this.setNextQuestion();
                });

            } else {
                alert('Answer neccessary');
            }
        },

        getSystemQuestions() {
            if (this.systemId !== null) {
                fetch(`/questions/${this.systemId}`, {
                    method: "get",
                    headers: { "Content-Type": "application/json" }
                })
                .then((res) => res.json())
                .then((data) => {
                    this.arrQuestions = data;
                    this.setNextQuestion();
                });
            }
        },

        getSystemElements() {
            if (this.systemId !== null) {
                fetch(`/elements/${this.systemId}`, {
                    method: "GET",
                    headers: { "Content-Type": "application/json" }
                })
                .then((res) => res.json())
                .then((data) => {
                    if (data.length > 1) {
                        this.arrElements = data;
                    } else {
                        alert('You need to create at least 2 elements');
                        window.location.href = '/';
                    }
                });
            }
        },

        getSystemTitle() {
            fetch(`/system/getTitle/${this.systemId}`, {
                method: "get",
                headers: { "Content-Type": "application/json" }
            })
            .then((res) => res.json())
            .then((data) => {
                this.systemTitle = data.title;
            });
        },

        setNextQuestion() {
            const randomIndex = helper.getRandomInt(this.arrQuestions.length);
            if (this.answerHistory.length > 0) {
                var count = 0;
                for (const i of this.answerHistory) {
                    if (this.arrQuestions[randomIndex].id === i.question.id) {
                        count++;
                    }
                }
                if (count === 0) {
                    this.form.question = this.arrQuestions[randomIndex];
                    this.form.answer = null;
                } else {
                    this.setNextQuestion();
                }
            } else {
                this.form.question = this.arrQuestions[randomIndex];
                this.form.answer = null;
            }
        },

        setSystemId() {
            this.systemId = helper.getRouteParam(1);
        },

        clearLocalStorageResult() {
            const result = JSON.parse(window.localStorage.getItem('result'));
            if (result !== null) {
                window.localStorage.removeItem('result');
            }
        }
    },
    mounted() {
        this.setSystemId();
        this.clearLocalStorageResult();
        this.getSystemQuestions();
        this.getSystemElements();
        this.getSystemTitle();
        this.modal = new bootstrap.Modal(document.getElementById('modalHelp'), {
            keyboard: false,
            backdrop: 'static'
        });
    }
});