<template>
  <div>
    <section class="out-section">
      <div class="container">
        <div class="section-heading">
          Вероятность усиления руки
        </div>
        <div class="instruction-heading">
          Выберите 5-6 карт (2 ручные карты и 3-4 на столе)
        </div>
        <div class="deck-wrapper">
          <ul class="ui-deck">
            <li
              v-for="(group, index) in deckGrouped"
              v-bind:key="`group-${index}`"
            >
              <ul class="ui-deck__group">
                <li
                  v-for="(card, index0) in group"
                  v-bind:key="`card-${index}-${index0}`"
                >
                  <div
                    class="card"
                    :class="[
                      card.selected ? 'selected' : '',
                      card.out ? 'out' : '',
                      card.selected || concatCards.length < 6
                        ? 'selectable'
                        : '',
                    ]"
                    @click="cardClick(card)"
                  >
                    <div>
                      <span>{{ card.value.toUpperCase() }}</span
                      ><span
                        :class="[
                          card.suit === 'h' || card.suit === 'd'
                            ? 'red-suit'
                            : '',
                          'suit',
                        ]"
                        >{{ suitToChar(card.suit) }}</span
                      >
                    </div>
                  </div>
                </li>
              </ul>
            </li>
          </ul>
        </div>
        <div class="message-container" v-if="notEnoughCards">
          <ul>
            <li v-if="notEnoughCards">
              <div class="message">
                <div class="message__icon">
                  <svg
                    xmlns="http://www.w3.org/2000/svg"
                    height="24px"
                    viewBox="0 0 24 24"
                    width="24px"
                  >
                    <path
                      d="M1 21h22L12 2 1 21zm12-3h-2v-2h2v2zm0-4h-2v-4h2v4z"
                    />
                  </svg>
                </div>
                <div class="message__text">
                  Выбрано недостаточно карт
                </div>
              </div>
            </li>
          </ul>
        </div>
        <div class="outs-summary" v-if="notEnoughCards === false">
          <div v-if="draws && draws.length">
            <div class="outs-heading">Дро-комбинации</div>
            <table class="draw-table">
              <tr v-for="(draw, index) in draws" v-bind:key="`draw-${index}`">
                <td>{{ draw.type === "flush" ? "Флеш" : "Стрит" }}</td>
                <td>
                  <ul class="draw-list">
                    <li
                      v-for="(card, index1) in draw.viewMask"
                      v-bind:key="`card-${index}-${index1}`"
                    >
                      <div :class="['mini-card', { out: card.out }]">
                        <div>
                          <span
                            :class="[{ straight: draw.type === 'straight' }]"
                            >{{ card.value.toUpperCase() }}</span
                          >
                          <span
                            v-if="card.suit !== ''"
                            :class="[
                              card.suit === 'h' || card.suit === 'd'
                                ? 'red-suit'
                                : '',
                              { flush: draw.type === 'flush' },
                              'suit',
                            ]"
                            >{{ suitToChar(card.suit) }}</span
                          >
                        </div>
                      </div>
                    </li>
                  </ul>
                </td>
              </tr>
            </table>
          </div>
          <div class="outs-heading">Итого</div>
          <div>
            Ауты:
            <span class="big-number text-out">{{ allOuts.length }}</span>
          </div>
          <div>
            Шансы:
            <span class="big-number text-out">{{ allOdds.ratio }}</span> или
            <span class="big-number text-out">{{ allOdds.perc }}%</span>
          </div>
        </div>
      </div>
    </section>
    <section class="pot-section">
      <div class="container"></div>
    </section>
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
    <!-- <div v-if="draws && draws.length">
      <div v-if="allOdds">Оддсы: {{ allOdds.ratio }}, {{ allOdds.perc }}%</div>
      <div v-if="potOdds">
        Пот-оддсы: {{ potOdds.ratio }}, {{ potOdds.perc }}%
      </div>
      <div v-if="typeof shouldCall === 'boolean'">
        Ставить? {{ shouldCall ? "Да" : "Нет" }}
      </div>
    </div> -->
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

function gcd(a, b) {
  let temp;
  let m;

  if (b > a) {
    temp = a;
    a = b;
    b = temp;
  }
  while (b != 0) {
    m = a % b;
    a = b;
    b = m;
  }
  return a;
}

function ratio(x, y) {
  const c = gcd(x, y);

  return `${x / c}:${y / c}`;
}

export default {
  data() {
    return {
      pot: 0,
      call: 0,
      concatCards: [],
    };
  },
  computed: {
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
    deckGrouped: function() {
      const temp0 = JSON.parse(JSON.stringify(this.deck));
      const highlighted = temp0.map((item) => {
        const temp = item;

        if (
          this.concatCards.length &&
          this.concatCards.filter(
            (item0) => item0.value === item.value && item0.suit === item.suit
          ).length
        ) {
          temp.selected = true;
        } else {
          temp.selected = false;
        }

        if (
          this.allOuts &&
          this.allOuts.length &&
          this.allOuts.filter(
            (item0) => item0.value === item.value && item0.suit === item.suit
          ).length
        ) {
          temp.out = true;
        } else {
          temp.out = false;
        }

        return temp;
      });

      return Object.values(
        highlighted.reduce(function(rv, x) {
          (rv[x["value"]] = rv[x["value"]] || []).push(x);
          return rv;
        }, {})
      ).sort((a, b) => a[0].valueWeight > b[0].valueWeight);
    },
    notEnoughCards: function() {
      return this.concatCards.length < 5;
    },
    unseenDeck: function() {
      return this.deck.filter(
        (item) =>
          this.concatCards.filter(
            (item0) => item0.suit === item.suit && item0.value === item.value
          ).length === 0
      );
    },
    unseenDeckGrouped: function() {
      return Object.values(
        this.unseenDeck.reduce(function(rv, x) {
          (rv[x["value"]] = rv[x["value"]] || []).push(x);
          return rv;
        }, {})
      ).sort((a, b) => a[0].valueWeight > b[0].valueWeight);
    },
    combAll: function() {
      const temp = [];

      if (
        this.unseenDeck &&
        this.concatCards.length >= 5 &&
        this.concatCards.length < 7
      ) {
        var groupBy = function(xs, key) {
          return xs.reduce(function(rv, x) {
            (rv[x[key]] = rv[x[key]] || []).push(x);
            return rv;
          }, {});
        };

        const tempFlush = [];

        Object.values(groupBy(this.concatCards, "suit")).forEach((item) => {
          item.forEach((item0, index, array) => {
            if (array.length - index >= 3) {
              const corrected =
                array.length - index >= 5 ? 5 : array.length - index;
              tempFlush.push(array.slice(index, index + corrected));
            }
          });
        });

        const flush = tempFlush.map((item) => {
          return {
            list: item,
            type: "flush",
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

          flush.forEach((item) => {
            temp.push(item);
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

          if (count > 3) {
            straights.push({
              list: item
                .filter((item0) =>
                  concatMappedByValue.some((item1) => item1 === item0.value)
                )
                .map((item) =>
                  this.concatCards.find((item1) => item1.value === item.value)
                ),
              mask: item,
              type: "straight",
            });
          }
        });

        if (straights.length) {
          straights.forEach((item) => {
            const outValues = item.mask.filter(
              (item1) =>
                item.list
                  .map((item2) => item2.value)
                  .some((item2) => item2 === item1.value) === false
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

          straights.forEach((item) => {
            temp.push(item);
          });
        }
      }

      return temp;
    },
    draws: function() {
      if (this.combAll.length) {
        let temp = JSON.parse(JSON.stringify(this.combAll));

        if (temp.filter((item) => item.list.length === 5).length) {
          if (temp.find((item0) => item0.list.length === 5).type === "flush") {
            temp = [];
          } else {
            temp = temp.filter(
              (item) =>
                item.type !== temp.find((item0) => item0.list.length === 5).type
            );
          }
        }

        temp = temp.filter(
          (item) => item.list.length >= (this.concatCards.length === 5 ? 3 : 4)
        );

        temp.forEach((item) => {
          if (temp.filter((item1) => item1.type === item.type).length > 1) {
            temp = temp.filter(
              (item2) =>
                !(
                  item2.type === item.type &&
                  item2.list.length !==
                    Math.max(
                      ...temp
                        .filter((item1) => item1.type === item.type)
                        .map((el) => el.list.length)
                    )
                )
            );
          }
        });

        temp.forEach((item, index) => {
          if (
            temp.filter(
              (item1) =>
                JSON.stringify(item1.outs) === JSON.stringify(item.outs)
            ).length > 1
          ) {
            const maxWeight = Math.max(
              ...temp
                .filter(
                  (item1) =>
                    JSON.stringify(item1.outs) === JSON.stringify(item.outs)
                )
                .map((item1) => item1.mask[4].valueWeight)
            );
            if (item.mask[4].valueWeight !== maxWeight) {
              temp = temp.filter((item1, index1) => index1 !== index);
            }
          }
        });

        return temp;
      }

      return null;
    },
    allOuts: function() {
      if (this.draws && this.draws.length) {
        let temp = [];

        this.draws.forEach((item) => {
          temp = temp.concat(item.outs);
        });

        return temp.filter(
          (v, i, a) =>
            a.findIndex((t) => t.suit === v.suit && t.value === v.value) === i
        );
      }

      return [];
    },
    allOdds: function() {
      if (this.allOuts) {
        return {
          ratio: ratio(this.allOuts.length, this.unseenDeck.length),
          perc: Math.round(
            (this.allOuts.length / this.unseenDeck.length) * 100
          ),
        };
      }
      return [];
    },
    potOdds: function() {
      if (this.call > 0 && this.pot > 0) {
        return {
          ratio: ratio(this.call, this.pot),
          perc: Math.round((this.call / this.pot) * 100),
        };
      }

      return null;
    },
    shouldCall: function() {
      if (this.draws && this.allOdds && this.potOdds) {
        if (this.allOdds.perc > this.potOdds.perc) {
          return true;
        }

        return false;
      }

      return null;
    },
  },
  methods: {
    suitToChar(suit) {
      return suit === "c"
        ? "♣"
        : suit === "d"
        ? "♦"
        : suit === "h"
        ? "♥"
        : suit === "s"
        ? "♠"
        : "";
    },
    cardClick(card) {
      if (
        this.concatCards.filter(
          (item) => item.value === card.value && item.suit === card.suit
        ).length
      ) {
        this.concatCards = this.concatCards.filter(
          (item) => !(item.value === card.value && item.suit === card.suit)
        );
      } else {
        if (this.concatCards.length < 6) {
          this.concatCards.push(
            this.deck.find(
              (item) => item.suit === card.suit && item.value === card.value
            )
          );
        }
      }
    },
  },
};
</script>

<style lang="scss">
@import "@/styles/variables";

.container {
  max-width: 1250px;
  padding: 0 15px;
  margin: 0 auto;
}

.out-section {
  padding: 3rem 0;
}

.deck-wrapper {
  display: flex;
  justify-content: center;
}

.ui-deck {
  display: flex;
  margin: 0 -5px;
}

.ui-deck > li {
  padding: 0 5px;
}

.ui-deck__group {
  display: flex;
  flex-direction: column;
  margin: -5px 0;
}

.ui-deck__group > li {
  padding: 5px 0;
}

.card {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 50px;
  height: (50px * 1.390625);
  border: 1px solid darken($background, 10%);
  border-radius: 5px;
  background-color: white;
  color: $textColor;
  font-family: "Old Standard TT", serif;
  font-size: 24px;
  user-select: none;
  transition: 0.1s linear;
  pointer-events: none;
}

.card.selectable {
  pointer-events: all;
  cursor: pointer;
}

.card.selectable:hover {
  border-color: darken($background, 30%);
}

.card.selected {
  border-top-width: 3px;
  border-bottom-width: 3px;
  border-color: $selected;
}

.card.selected:hover {
  border-color: darken($selected, 10%);
}

.card.out {
  border-top-width: 3px;
  border-bottom-width: 3px;
  border-color: $out;
}

.card.out:hover {
  border-color: darken($out, 30%);
}

.card .suit {
  font-size: 1.2em;
}

.red-suit {
  color: $redSuit;
}

.section-heading {
  margin-bottom: 1.5em;
  font-size: 20px;
  text-align: center;
  font-weight: bold;
  color: $textColor;
}

.instruction-heading {
  margin-bottom: 20px;
  font-size: 14px;
  text-align: center;
  font-weight: bold;
  color: $textColor;
}

.message-container {
  display: flex;
  justify-content: center;
  margin: 20px 0;
}

.message {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 400px;
  padding: 1em;
  background-color: $warning;
  border-radius: 10px;
  font-size: 14px;
}

.message__icon {
  flex: 0 0 35px;
  max-width: 35px;
}

.message__icon svg {
  display: block;
  width: 25px;
  height: 25px;
}

.outs-summary {
  max-width: 400px;
  margin: 20px auto 0;
  font-size: 14px;
  line-height: 1.4;
  color: $textColor;
}

* + .outs-heading {
  margin-top: 1em;
}

.outs-heading {
  margin-bottom: 0.5em;
  font-weight: bold;
}

.text-selected {
  color: $selected;
}

.text-out {
  color: $out;
}

.big-number {
  font-size: 2em;
}

.mini-card {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 30px;
  height: 30px;
  border: 1px solid $selected;
}

.mini-card.out {
  border-color: $out;
}

.mini-card .straight,
.mini-card .flush {
  font-weight: bold;
}

.mini-card .suit {
  font-size: 1.2em;
}

.draw-table td {
  padding: 3px 0;
  vertical-align: baseline;
}

.draw-table td:first-child {
  padding-right: 6px;
}

.draw-list {
  display: flex;
}

.draw-list li {
  padding: 0 3px;
}
</style>
