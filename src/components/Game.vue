<script>
import "../style/game.css"
export default {
    data() {
        return {
            money: 3000,
            stability: 50,
            workers: 1,
            demand: 0,
            card1: null,
            card2: null,
            card3: null,
            chosenCard: null,
            event: null,
            confirmDialog: false,
            rounds: 1,
        };
    },
    computed: {
        formattedMoney() {
            return new Intl.NumberFormat('pt-BR', {
                style: 'currency',
                currency: 'BRL',
            }).format(this.money);
        },
    },
    methods: {
        gameOver() {
            window.location.href = "/"
        },
        verify() {
            if (this.stability > 100) this.stability = 100;
            if (this.money < 0) this.stability--;
            if (this.demand < 0) this.demand = 0;
            if (this.workers * 3 < this.demand) this.stability -= 5;

            if (this.money < -5000) {
                this.gameOver();
            }

            if (this.workers < 1) {
                this.gameOver();
            }

            if (this.stability < 1) {
                this.gameOver();
            }

            if (this.money < 0 && this.stability < 50) {
                this.gameOver();
            }
        },

        nextRandom(min, max) {
            return Math.floor(Math.random() * (max - min + 1)) + min;
        },
        pickUniqueCards() {
            const availableActions = [...actions];
            const selectedCards = [];

            for (let i = 0; i < 3; i++) {
                const randomIndex = this.nextRandom(0, availableActions.length - 1);
                const action = availableActions.splice(randomIndex, 1)[0];
                selectedCards.push(action);
            }

            const eventDrop = this.nextRandom(1, 100);

            if (eventDrop > 90) {
                const randomIndex = this.nextRandom(0, randomEvents.length - 1);
                this.event = randomEvents[randomIndex];
                this.showEventDialog();
            } else {
                this.event = null;
            }

            this.card1 = selectedCards[0];
            this.card2 = selectedCards[1];
            this.card3 = selectedCards[2];
        },

        showEventDialog() {
            const dialog = this.$refs.eventDialog;
            if (dialog) dialog.style.display = "block";
        },

        closeEventDialog() {
            const dialog = this.$refs.eventDialog;
            if (dialog) dialog.style.display = "none";
        },

        handleEventDecision(decision) {
            if (this.event) {
                const choice = this.event[decision];
                this.money += choice.dinheiro;
                this.stability += choice.estabilidade;
                this.workers += choice.trabalhadores;
                this.demand += choice.demanda;

                this.verify();
                this.closeEventDialog();
                this.refresh();
            }
        },
        showConfirmDialog(card) {
            this.chosenCard = card;
            const dialog = this.$refs.confirmDialog;
            if (dialog) dialog.style.display = "block";
        },
        closeDialog() {
            const dialog = this.$refs.confirmDialog;
            if (dialog) dialog.style.display = "none";
        },
        submit() {
            if (this.chosenCard) {
                this.money += this.chosenCard.dinheiro;
                this.stability += this.chosenCard.estabilidade;
                this.workers += this.chosenCard.trabalhadores;
                this.demand += this.chosenCard.demanda;
                this.rounds++;

                this.verify();

                this.money -= this.workers * 100;
                this.money += this.demand * 50;

                this.closeDialog();
                this.refresh();
            }
        },
        refresh() {
            this.pickUniqueCards();
        }
    },
    created() {
        this.pickUniqueCards();
    },
};

const actions = [
    {
        id: 1,
        acao: "Contratar novo funcionário",
        dinheiro: -1500,
        estabilidade: 5,
        trabalhadores: 1,
        demanda: 0,
    },
    {
        id: 2,
        acao: "Aceitar trabalho",
        dinheiro: 0,
        estabilidade: -3,
        trabalhadores: 0,
        demanda: 5,
    },
    {
        id: 3,
        acao: "Fazer parceria",
        dinheiro: 500,
        estabilidade: -10,
        trabalhadores: 0,
        demanda: 3,
    },
    {
        id: 4,
        acao: "Dar aumento aos trabalhadores",
        dinheiro: -100,
        estabilidade: 8,
        trabalhadores: 0,
        demanda: 0,
    },
    {
        id: 5,
        acao: "Melhorar o ambiente de trabalho",
        dinheiro: -1000,
        estabilidade: 10,
        trabalhadores: 0,
        demanda: 0,
    },
    {
        id: 6,
        acao: "Demitir funcionário",
        dinheiro: 1500,
        estabilidade: -10,
        trabalhadores: -1,
        demanda: 0,
    },
];

const randomEvents = [
    {
        id: 1,
        evento: "Um bug foi encontrado",
        solucionar: {
            dinheiro: -500,
            estabilidade: 5,
            trabalhadores: 0,
            demanda: 0,
        },
        postergar:
        {
            dinheiro: 0,
            estabilidade: -5,
            trabalhadores: 0,
            demanda: 5,
        }
    },
    {
        id: 2,
        evento: "Um trabalhador foi pego dormindo no trabalho",
        solucionar: {
            dinheiro: 0,
            estabilidade: 10,
            trabalhadores: -1,
            demanda: 0,
        },
        postergar:
        {
            dinheiro: 0,
            estabilidade: -5,
            trabalhadores: 0,
            demanda: 5,
        }
    },
    {
        id: 3,
        evento: "O cliente quer adicionar algo no meio da sprint",
        solucionar: {
            dinheiro: 400,
            estabilidade: 10,
            trabalhadores: 0,
            demanda: 5,
        },
        postergar:
        {
            dinheiro: 0,
            estabilidade: -10,
            trabalhadores: 0,
            demanda: 0,
        }
    }
]

</script>

<template>
    <h4 style="position:absolute; top:3%; left:15%">Round: {{ rounds }}</h4>
    <div style="display: flex; flex-direction: row; justify-content: center; gap: 50px">
        <h4>Dinheiro: {{ formattedMoney }}</h4>
        <h4>Estabilidade: {{ stability }}%</h4>
    </div>
    <div style="display: flex; flex-direction: row; justify-content: center; gap: 50px">
        <h4>Funcionários: {{ workers }}</h4>
        <h4>Demanda: {{ demand }}</h4>
    </div>
    <div id="buttonArea" ref="choiceButtons">
        <button v-if="this.card1" @click="showConfirmDialog(this.card1)">
            <h4>{{ card1.acao }}</h4>
            <p>Dinheiro: {{ card1.dinheiro }}</p>
            <p>Estabilidade: {{ card1.estabilidade }}</p>
            <p>Funcionários: {{ card1.trabalhadores }}</p>
            <p>Demanda: {{ card1.demanda }}</p>
        </button>
        <button v-if="this.card2" @click="showConfirmDialog(this.card2)">
            <h4>{{ card2.acao }}</h4>
            <p>Dinheiro: {{ card2.dinheiro }}</p>
            <p>Estabilidade: {{ card2.estabilidade }}</p>
            <p>Funcionários: {{ card2.trabalhadores }}</p>
            <p>Demanda: {{ card2.demanda }}</p>
        </button>
        <button v-if="this.card3" @click="showConfirmDialog(this.card3)">
            <h4>{{ card3.acao }}</h4>
            <p>Dinheiro: {{ card3.dinheiro }}</p>
            <p>Estabilidade: {{ card3.estabilidade }}</p>
            <p>Funcionários: {{ card3.trabalhadores }}</p>
            <p>Demanda: {{ card3.demanda }}</p>
        </button>
    </div>

    <div id="confirmDialog" ref="confirmDialog" style="display: none">
        <h4>Confirma sua decisão?</h4>
        <div id="confirmButtons">
            <button id="cancel" @click="closeDialog">Não</button>
            <button id="confirm" @click="submit">Sim</button>
        </div>
    </div>

    <div id="eventDialog" v-if="event" ref="eventDialog">
        <h2>{{ event.evento }}</h2>
        <h4>Qual sua decisão?</h4>
        <div id="decision">
            <div id="postergar">
                <button @click="handleEventDecision('postergar')">Postergar</button>
                <p>Dinheiro: {{ event.postergar.dinheiro }}</p>
                <p>Estabilidade: {{ event.postergar.estabilidade }}</p>
                <p>Funcionários: {{ event.postergar.trabalhadores }}</p>
                <p>Demanda: {{ event.postergar.demanda }}</p>
            </div>
            <div id="resolver">
                <button @click="handleEventDecision('solucionar')">Resolver</button>
                <p>Dinheiro: {{ event.solucionar.dinheiro }}</p>
                <p>Estabilidade: {{ event.solucionar.estabilidade }}</p>
                <p>Funcionários: {{ event.solucionar.trabalhadores }}</p>
                <p>Demanda: {{ event.solucionar.demanda }}</p>
            </div>
        </div>
    </div>

</template>
