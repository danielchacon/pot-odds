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
        <li
          v-for="(draw, index) in draws"
          v-bind:key="`out-${index}`"
          style="padding: 5px 0;"
        >
          <div>{{ draw.name }}</div>
          <ul>
            <li
              v-for="(group, index0) in draw.got"
              v-bind:key="`got-${index}-${index0}`"
              style="padding: 5px 0;"
            >
              <ul style="display: flex; flex-wrap: wrap;">
                <li
                  v-for="(card, index1) in group.viewMask"
                  v-bind:key="`card-${index}-${index0}-${index1}`"
                >
                  <div
                    :style="(card.out ? 'opacity: 0.6;' : '') + 'width: 35px;'"
                  >
                    <span :style="draw.type === 'straight' ? 'font-weight: bold;' : ''">{{ card.value.toUpperCase() }}</span>
                    <span
                      v-if="card.suit !== ''"
                      :style="
                        (card.suit === 'h' || card.suit === 'd'
                          ? 'color: red;'
                          : '') + (draw.type === 'flush' ? 'font-weight: bold;' : '')
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
      <div v-if="allOuts">Ауты: {{ allOuts.length }}</div>
      <ul v-if="allOuts">
        <li
          v-for="(group, index) in unseenDeckGrouped"
          v-bind:key="`group-${index}`"
        >
          <ul style="display: flex; flex-wrap: wrap;">
            <li
              v-for="(card, index0) in group"
              v-bind:key="`card-${index}-${index0}`"
              style="padding: 5px 0;"
            >
              <div
                :style="
                  (allOuts.filter(
                    (item) =>
                      item.suit === card.suit && item.value === card.value
                  ).length
                    ? 'font-weight: bold;'
                    : 'opacity: 0.6;') + 'width: 35px;'
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
      <div v-if="allOdds">Оддсы: {{ allOdds }}%</div>
      <div v-if="potOdds">Пот-оддсы: {{ potOdds }}%</div>
      <div v-if="typeof shouldCall === 'boolean'">
        Ставить? {{ shouldCall ? "Да" : "Нет" }}
      </div>
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
      hand: "2d3d",
      board: "4h5dqh",
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
                (item0) =>
                  item.list.filter((item1) => item1.suit === item0.suit)
                    .length === 0
              )
            );
          });

          flush.forEach((item) => {
            item.viewMask = item.list.concat(
              Array.from({ length: 5 - item.list.length }, () => ({
                suit: item.list[0].suit,
                value: "",
                out: true,
              }))
            );
          });

          temp.push({
            name: "Флеш-дро",
            type: "flush",
            got: flush,
          });
        }

        const straightDeck = JSON.parse(
          JSON.stringify(
            this.deck
              .map((item) => ({
                value: item.value,
                valueWeight: item.valueWeight,
              }))
              .slice(0, 13)
          )
        );

        straightDeck.unshift({
          value: this.deck[this.deck.length - 1].value,
          valueWeight: 0,
        });

        const straightMasks = [];

        straightDeck.forEach((item, index, array) => {
          if (index < straightDeck.length - 4) {
            straightMasks.push(array.slice(index, index + 5));
          }
        });

        const straights = [];
        const concatMappedByValue = this.concatCards.map(
          (item0) => item0.value
        );

        straightMasks.forEach((item) => {
          let count = 0;

          item.forEach((item0) => {
            if (
              concatMappedByValue.filter((item1) => item1 === item0.value)
                .length
            ) {
              count += 1;
            }
          });

          if (count >= 3) {
            straights.push({
              list: item
                .filter((item0) =>
                  concatMappedByValue.some((item1) => item1 === item0.value)
                )
                .map((item) =>
                  this.concatCards.find((item1) => item1.value === item.value)
                ),
              mask: item,
            });
          }
        });

        if (straights.length) {
          straights.forEach((item) => {
            const outValues = item.mask.filter(
              (item1) =>
                item.list
                  .map((item2) => item2.value)
                  .some((item2) => item2 === item1) === false
            );

            item.outs = this.unseenDeck.filter(
              (item0) =>
                outValues.filter((item1) => item1.value === item0.value).length
            );
          });

          straights.forEach((item) => {
            const temp1 = JSON.parse(JSON.stringify(this.concatCards));

            temp1.sort((a, b) => a.valueWeight > b.valueWeight);

            item.concatCardsSorted = temp1;
          });

          straights.forEach((item) => {
            const temp1 = item.list.concat(
              item.mask
                .filter(
                  (item0) =>
                    item.list.some((item1) => item1.value === item0.value) ===
                    false
                )
                .map((item0) => ({
                  suit: "",
                  value: item0.value,
                  valueWeight: item0.valueWeight,
                  out: true,
                }))
            );

            temp1.sort((a, b) => a.valueWeight > b.valueWeight);

            item.viewMask = temp1;
          });

          if (
            straights.filter((item) => item.list.length === item.mask.length)
              .length === 0
          ) {
            const got = straights.filter(
              (item) =>
                item.list.length >=
                Math.max(...straights.map((el) => el.list.length))
            );
            temp.push({
              name: "Стрит-дро",
              type: "straight",
              got,
            });
          }
        }

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
      if (this.allOuts) {
        return Math.round(
          (this.allOuts.length / this.unseenDeck.length +
            this.allOuts.length / (this.unseenDeck.length - 1) -
            ((this.allOuts.length / this.unseenDeck.length) *
              (this.allOuts.length - 1)) /
              (this.unseenDeck.length - 1)) *
            100
        );
      }
      return null;
    },
    potOdds: function() {
      if (this.call > 0 && this.pot > 0) {
        return Math.round((this.call / this.pot) * 100);
      }

      return null;
    },
    shouldCall: function() {
      if (this.draws && this.allOdds && this.potOdds) {
        if (this.allOdds > this.potOdds) {
          return true;
        }

        return false;
      }

      return null;
    },
  },
  methods: {
    inputSplitted(inputVal) {
      const bp = [0];

      inputVal.split("").forEach((item, index, array) => {
        if (suits.filter((item0) => item0 === item).length) {
          if (index + 1 !== array.length) {
            bp.push(index + 1);
          }
        }
      });

      const intervals = bp.map((item, index) => ({
        startIndex: item,
        length:
          item - bp[index + 1] ? bp[index + 1] - item : inputVal.length - item,
      }));

      const splitted = [];

      intervals.forEach((item) => {
        splitted.push(inputVal.substr(item.startIndex, item.length));
      });

      const temp = splitted.map((item) =>
        this.deck.find(
          (item0) =>
            item0.suit === item.slice(item.length - 1, item.length) &&
            item0.value === item.slice(0, item.length - 1)
        )
      );

      return temp;
    },
    inputIsValid(inputVal, type) {
      if (inputVal.length) {
        let validCount = 0;
        const thisSplit = this.inputSplitted(inputVal);

        thisSplit.forEach((item) => {
          if (
            item.suit.length &&
            item.value.length &&
            suits.filter((item0) => item0.suit === item.suit).length === 0 &&
            values.filter((item0) => item0.value === item.value).length === 0
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

      return true;
    },
  },
};

// TODO: общая таблица всех карт, выбор карт кликом, выбранные карты меняют обводку или фон, в этой же таблице ауты показываем, убираем текстовые инпуты
</script>
