<template>
  <div style="padding: 10px;">
    <div style="padding: 5px 0;">
      Рука:
      <input
        type="text"
        v-model="hand"
        style="width: 150px; border-bottom: 1px solid black;"
      />
      <div v-if="!handValid">Ошибка</div>
    </div>
    <div style="padding: 5px 0;">
      Стол:
      <input
        type="text"
        v-model="board"
        style="width: 150px; border-bottom: 1px solid black;"
      />
      <div v-if="!boardValid">Ошибка</div>
    </div>
    <div style="padding: 5px 0;">
      Кон:
      <input
        type="number"
        v-model="pot"
        style="width: 150px; border-bottom: 1px solid black;"
      />
    </div>
    <div style="padding: 5px 0;">
      Колл:
      <input
        type="number"
        v-model="call"
        style="width: 150px; border-bottom: 1px solid black;"
      />
    </div>
    <div v-if="draws">
      <br />
      <ul>
        <li v-for="(draw, index) in draws" v-bind:key="`out-${index}`">
          <div>{{ draw.name }}</div>
          <ul>
            <li
              v-for="(group, index0) in draw.got"
              v-bind:key="`got-${index}-${index0}`"
            >
              <ul style="display: flex; flex-wrap: wrap;">
                <li
                  v-for="(card, index1) in group.concatCardsSorted"
                  v-bind:key="`card-${index}-${index0}-${index1}`"
                  style="padding: 5px;"
                >
                  <div
                    :style="
                      group.list.filter(
                        (item) =>
                          item.suit === card.suit && item.value === card.value
                      ).length
                        ? 'font-weight: bold'
                        : 'opacity: 0.6'
                    "
                  >
                    {{ card.value.toUpperCase()
                    }}<span
                      :style="
                        card.suit === 'h' || card.suit === 'd'
                          ? 'color: red;'
                          : ''
                      "
                      >{{ card.suit.toUpperCase() }}</span
                    >
                  </div>
                </li>
              </ul>
            </li>
          </ul>
        </li>
      </ul>
      <br />
      <div>Ауты: {{ allOuts.length }}</div>
      <ul>
        <li
          v-for="(group, index) in unseenDeckGrouped"
          v-bind:key="`group-${index}`"
        >
          <ul style="display: flex; flex-wrap: wrap;">
            <li
              v-for="(card, index0) in group"
              v-bind:key="`card-${index}-${index0}`"
              style="padding: 5px;"
            >
              <div
                :style="
                  allOuts.filter(
                    (item) =>
                      item.suit === card.suit && item.value === card.value
                  ).length
                    ? 'font-weight: bold'
                    : 'opacity: 0.6'
                "
              >
                {{ card.value.toUpperCase()
                }}<span
                  :style="
                    card.suit === 'h' || card.suit === 'd' ? 'color: red;' : ''
                  "
                  >{{ card.suit.toUpperCase() }}</span
                >
              </div>
            </li>
          </ul>
        </li>
      </ul>
      <br />
      <div>Оддсы: {{ allOdds }}%</div>
      <div>Пот-оддсы: {{ potOdds }}%</div>
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

          if (parseInt(values[x], 10) <= 10) {
            card.valueWeight = parseInt(values[x], 10);
          }

          if (values[x] === "j") {
            card.valueWeight = 11;
          }

          if (values[x] === "q") {
            card.valueWeight = 12;
          }

          if (values[x] === "k") {
            card.valueWeight = 13;
          }

          if (values[x] === "a") {
            card.valueWeight = 14;
          }

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
    unseenDeckGrouped: function() {
      return Object.values(
        this.unseenDeck.reduce(function(rv, x) {
          (rv[x["value"]] = rv[x["value"]] || []).push(x);
          return rv;
        }, {})
      ).sort((a, b) => a[0].valueWeight > b[0].valueWeight);
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
          .filter((item) => item.length >= 3 && item.length <= 5)
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
            item.concatCardsSorted = item.list.concat(
              this.concatCards.filter(
                (item0) => item.list.filter((item1) => item1.suit === item0.suit).length === 0
              )
            );
          });

          temp.push({
            name: "Флеш-дро",
            got: flush,
          });
        }

        const straightDeck = JSON.parse(
          JSON.stringify(this.deck.map((item) => item.value).slice(0, 13))
        );

        straightDeck.unshift(this.deck[this.deck.length - 1].value);

        // const straight = []

        console.log(straightDeck);

        return temp;
      }

      return null;
    },
    allOuts: function() {
      if (this.draws.length) {
        let temp = [];

        this.draws.forEach((item) => {
          item.got.forEach((item0) => {
            temp = temp.concat(item0.outs);
          });
        });

        return temp.filter(
          (v, i, a) =>
            a.findIndex((t) => t.suit === v.suit && t.value === v.value) === i
        );
      }

      return null;
    },
    allOdds: function() {
      return Math.round(
        (this.allOuts.length / this.unseenDeck.length +
          this.allOuts.length / (this.unseenDeck.length - 1) -
          ((this.allOuts.length / this.unseenDeck.length) *
            (this.allOuts.length - 1)) /
            (this.unseenDeck.length - 1)) *
          100
      );
    },
    potOdds: function() {
      return Math.round((this.call / this.pot) * 100);
    },
    shouldCall: function() {
      if (this.draws && this.allOdds > this.potOdds) {
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
