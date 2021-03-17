<template>
  <div>
    <div>
      Рука: <input type="text" v-model="hand" />
      <div v-if="!handValid">Ошибка</div>
    </div>
    <div>
      Стол: <input type="text" v-model="board" />
      <div v-if="!boardValid">Ошибка</div>
    </div>
    <div>Кон: <input type="number" v-model="pot" /></div>
    <div>Колл: <input type="number" v-model="call" /></div>
    <div>Пот-оддсы: {{ potOdds }}%</div>
    <div v-if="draws">
      <div>Дро:</div>
      <ul>
        <li v-for="(draw, index) in draws" v-bind:key="`out-${index}`">
          <div>{{ draw.name }}</div>
          <ul>
            <li
              v-for="(group, index0) in draw.got"
              v-bind:key="`got-${index}-${index0}`"
            >
              <div>В наличии:</div>
              <ul>
                <li
                  v-for="(card, index1) in group.list"
                  v-bind:key="`card-${index}-${index0}-${index1}`"
                >
                  {{ card.value + card.suit }}
                </li>
              </ul>
              <div>Ауты: {{ group.outs.length }}</div>
              <div>Оддсы: {{ group.fillOdds }}%</div>
            </li>
          </ul>
        </li>
      </ul>
      <div>Ставить? {{ shouldCall ? "Да" : "Нет" }}</div>
    </div>
  </div>
</template>

<script>
const suits = ["s", "d", "c", "h"];
const values = [
  "2",
  "3",
  "4",
  "5",
  "6",
  "7",
  "8",
  "9",
  "10",
  "j",
  "q",
  "k",
  "a",
];

export default {
  data() {
    return {
      hand: "jh 2s",
      board: "3h js 10s",
      pot: 450,
      call: 70,
    };
  },
  computed: {
    handValid: function() {
      return this.inputIsValid(this.hand.trim().toLowerCase(), "hand");
    },
    boardValid: function() {
      return this.inputIsValid(this.board.trim().toLowerCase(), "board");
    },
    concatCards: function() {
      const hand = this.inputSplitted(this.hand.trim().toLowerCase());
      const board = this.inputSplitted(this.board.trim().toLowerCase());

      return hand.concat(board);
    },
    deck: function() {
      var deck = new Array();

      for (var i = 0; i < suits.length; i++) {
        for (var x = 0; x < values.length; x++) {
          var card = { value: values[x], suit: suits[i] };
          deck.push(card);
        }
      }

      return deck;
    },
    unseenDeck: function() {
      if (this.handValid && this.boardValid) {
        return this.deck.filter(
          (item) =>
            this.concatCards.filter(
              (item0) => item0.suit === item.suit && item0.value === item.value
            ).length === 0
        );
      }

      return null;
    },
    draws: function() {
      if (this.unseenDeck) {
        const temp = [];

        var groupBy = function(xs, key) {
          return xs.reduce(function(rv, x) {
            (rv[x[key]] = rv[x[key]] || []).push(x);
            return rv;
          }, {});
        };

        const flush = Object.values(groupBy(this.concatCards, "suit"))
          .filter((item) => item.length > 1 && item.length <= 5)
          .map((item) => {
            return {
              list: item,
            };
          });

        if (flush.length) {
          flush.forEach((item) => {
            item.outs = this.unseenDeck
              .filter(
                (item1) =>
                  item.list.filter(
                    (item0) =>
                      item0.suit === item1.suit && item0.value === item1.value
                  ).length === 0
              )
              .filter((item1) => item1.suit === item.list[0].suit);
          });

          flush.forEach((item) => {
            item.fillOdds = Math.round(
              (item.outs.length / this.unseenDeck.length +
                item.outs.length / (this.unseenDeck.length - 1) -
                ((item.outs.length / this.unseenDeck.length) *
                  (item.outs.length - 1)) /
                  (this.unseenDeck.length - 1)) *
                100
            );
          });

          temp.push({
            name: "Флеш",
            got: flush,
          });
        }

        const straightDeck = JSON.parse(JSON.stringify(this.deck.map(item => item.value).slice(0, 13)));

        straightDeck.unshift(this.deck[this.deck.length - 1].value);

        // const straight = []

        console.log(straightDeck);

        return temp;
      }

      return null;
    },
    potOdds: function() {
      return Math.round((this.call / this.pot) * 100);
    },
    shouldCall: function() {
      if (
        this.draws &&
        this.draws.filter(
          (item) =>
            item.got.filter((item0) => item0.fillOdds > this.potOdds).length
        ).length
      ) {
        return true;
      }

      return false;
    },
  },
  methods: {
    inputSplitted(inputVal) {
      return inputVal.split(" ").map((item) => ({
        suit: item.slice(item.length - 1, item.length),
        value: item.slice(0, item.length - 1),
      }));
    },
    inputIsValid(inputVal, type) {
      if (inputVal.length) {
        let validCount = 0;
        const thisSplit = inputVal.split(" ");

        thisSplit.forEach((item) => {
          if (
            item.length > 1 &&
            suits.filter(
              (item0) => item0 === item.slice(item.length - 1, item.length)
            ).length &&
            values.filter((item0) => item0 === item.slice(0, item.length - 1))
              .length
          ) {
            validCount += 1;
          }
        });

        if (
          validCount === thisSplit.length &&
          this.concatCards.every(
            (e, i, a) =>
              a.findIndex(
                (item) => item.suit === e.suit && item.value === e.value
              ) === i
          )
        ) {
          if (type === "hand" && thisSplit.length === 2) {
            return true;
          }

          if (
            type === "board" &&
            thisSplit.length >= 3 &&
            thisSplit.length <= 5
          ) {
            return true;
          }
          return false;
        }

        return false;
      }

      return false;
    },
  },
};
</script>
