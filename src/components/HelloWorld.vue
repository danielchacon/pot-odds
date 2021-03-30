<template>
  <div>
    <header class="site-header">
      <div class="logo">
        <img src="@/assets/logo.png" srcset="@/assets/logo@2x.png 2x" />
      </div>
      <h1 class="site-heading">
        <div>Шансы на улучшение и шансы банка в покере:</div>
        <div>калькулятор</div>
      </h1>
    </header>
    <section class="out-section">
      <div class="container">
        <h2 class="section-heading">Шансы на улучшение</h2>
        <div class="section-text">
          <p>
            <strong>Ауты в покере</strong> — карты, которые могут улучшить руку
            до выигрышной.
          </p>
          <p>
            В нашем случае — все карты, которые на оставшихся раундах составят
            стрит или флеш.
          </p>
          <p>
            Неполные комбинации карт называются
            <strong>дро-комбинации</strong> (отсюда названия
            <strong>стрит-дро</strong> и <strong>флеш-дро</strong>).
          </p>
          <p>
            <strong>Как считать ауты в покере?</strong> Определяем каких карт не
            хватает до составления стрита или флеша, и подсчитываем сколько
            таких карт осталось среди неизвестных нам.
          </p>
          <p>
            <strong>Шансы на улучшение</strong> — это шансы составить полную
            комбинацию. Рассчитывается как соотношение аутов и неизвестных карт.
          </p>
          <p class="text-center">
            <span class="formula">A : B</span> или
            <span class="formula">(A / B) * 100%</span>
          </p>
          <p>
            <strong>Калькулятор аутов в покере</strong> поможет наглядно
            разобраться с подсчетами.
          </p>
        </div>
        <div class="deck-tools">
          <ul class="switcher">
            <li :class="[{ active: myCards }, 'my']" @click="myCards = true">
              Свои карты
            </li>
            <li :class="[{ active: !myCards }, 'opp']" @click="myCards = false">
              Карты оппонентов
            </li>
          </ul>
        </div>
        <div class="instruction-heading" v-if="myCards">
          Выберите 5-6 карт (2 в руке и 3-4 на столе)
        </div>
        <div class="instruction-heading" v-if="!myCards">
          Выберите предполагаемые карты оппонентов, тем самым исключив их из
          расчета
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
                      card.opp ? 'opp' : '',
                      (card.selected && myCards) ||
                      (concatCards.length < 6 && myCards) ||
                      (!myCards && !card.selected)
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
        <div class="deck-legend-wrapper">
          <ul class="deck-legend">
            <li>
              <span class="deck-legend__box deck-legend__box--hand"></span> —
              ваши карты
            </li>
            <li>
              <span class="deck-legend__box deck-legend__box--opp"></span> —
              чужие карты
            </li>
            <li>
              <span class="deck-legend__box deck-legend__box--outs"></span> —
              ауты
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
                <div class="message__text">Выбрано недостаточно карт</div>
              </div>
            </li>
          </ul>
        </div>
        <div
          class="outs-summary"
          v-if="
            notEnoughCards === false &&
              (completed.length || (draws && draws.length))
          "
        >
          <div class="outs-summary__section" v-if="completed.length">
            <div class="outs-heading">Полные комбинации</div>
            <table class="draw-table">
              <tbody>
                <tr
                  v-for="(comb, index) in completed"
                  v-bind:key="`comb-${index}`"
                >
                  <td>{{ comb.type === "flush" ? "Флеш" : "Стрит" }}</td>
                  <td>
                    <ul class="draw-list">
                      <li
                        v-for="(card, index1) in comb.viewMask"
                        v-bind:key="`card-${index}-${index1}`"
                      >
                        <div class="mini-card">
                          <div>
                            <span
                              :class="[{ straight: comb.type === 'straight' }]"
                              >{{ card.value.toUpperCase() }}</span
                            >
                            <span
                              v-if="card.suit !== ''"
                              :class="[
                                card.suit === 'h' || card.suit === 'd'
                                  ? 'red-suit'
                                  : '',
                                { flush: comb.type === 'flush' },
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
              </tbody>
            </table>
          </div>
          <div class="outs-summary__section" v-if="draws && draws.length">
            <div class="outs-heading">Дро-комбинации</div>
            <table class="draw-table">
              <thead>
                <tr>
                  <th colspan="2"></th>
                  <th>Ауты</th>
                  <th>Шансы</th>
                </tr>
              </thead>
              <tbody>
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
                  <td class="td-outs">
                    <span class="text-out"
                      >{{ draw.outs.length
                      }}<span class="mobile-only"> аутов</span></span
                    >
                  </td>
                  <td class="td-odds">
                    <span class="text-out"
                      >{{ draw.odds.ratio }} или {{ draw.odds.perc }}%</span
                    >
                  </td>
                </tr>
              </tbody>
            </table>

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
          <div class="outs-summary__section" v-else>
            <div class="outs-heading outs-heading--single">
              Нет дро-комбинаций
            </div>
          </div>
        </div>
        <div
          class="outs-summary"
          v-if="
            notEnoughCards === false &&
              completed.length === 0 &&
              (draws === null || draws.length === 0)
          "
        >
          <div class="outs-heading outs-heading--single">
            У вас нет подходящих комбинаций
          </div>
        </div>
      </div>
    </section>
    <section class="pot-section">
      <div class="container">
        <h2 class="section-heading">Шансы банка</h2>
        <div class="section-text">
          <p>
            <strong>Шансы банка в покере</strong> — это соотношение коллируемой
            ставки и размера банка.
          </p>
          <p class="text-center">
            <span class="formula">A : B</span> или
            <span class="formula">(A / (B + A)) * 100%</span>
          </p>
          <p>Ниже представлен <strong>калькулятор шансов банка</strong>.</p>
        </div>
        <div class="instruction-heading">
          Укажите сумму ставки и сумму в банке
        </div>
        <div class="pot-form">
          <div class="pot-form__list">
            <div class="pot-form__cell">
              <div class="text-field-wrapper">
                <label class="text-field-label">Ставка</label>
                <input
                  type="text"
                  v-model.number="call"
                  class="text-field"
                  @keypress="isNumber($event)"
                />
              </div>
            </div>
            <div class="pot-form__cell">
              <div class="text-field-wrapper">
                <label class="text-field-label">Банк</label>
                <input
                  type="text"
                  v-model.number="pot"
                  class="text-field"
                  @keypress="isNumber($event)"
                />
              </div>
            </div>
          </div>
        </div>
        <div class="message-container" v-if="potOdds === null">
          <ul>
            <li>
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
                  Укажите корректные размеры ставки и банка
                </div>
              </div>
            </li>
          </ul>
        </div>
        <div class="pot-results" v-if="potOdds">
          Шансы <span class="big-number text-out">{{ potOdds.ratio }}</span> или
          <span class="big-number text-out">{{ potOdds.perc }}%</span>
        </div>
      </div>
    </section>
    <section
      class="prediction-section"
      id="prediction"
      v-observe-visibility="visibilityChanged"
    >
      <div class="container">
        <div class="section-heading">Рекомендация</div>
        <div class="section-text">
          <p>
            Знание <strong>шансов на улучшение</strong> и
            <strong>шансов банка</strong> позволяет принять решение о выгодности
            ставки.
          </p>
          <p>
            Считается, что ставка выгодна, если
            <strong>шансы на улучшение</strong> выше
            <strong>шансов банка</strong>.
          </p>
        </div>
        <div
          class="message-container"
          v-if="!(potOdds && draws && draws.length)"
        >
          <ul>
            <li>
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
                <div
                  class="message__text"
                  v-if="
                    !(potOdds && draws && draws.length) &&
                      completed.length === 0
                  "
                >
                  Произведите расчет шансов на улучшение и банка
                </div>
                <div class="message__text" v-if="completed.length > 0">
                  Нет дро-комбинаций, расчет не требуется
                </div>
              </div>
            </li>
          </ul>
        </div>
        <div v-if="potOdds && draws && draws.length">
          <table class="prediction-table">
            <tbody>
              <tr>
                <td>Шансы на улучшение</td>
                <td>
                  <span class="text-out"
                    ><span class="big-number">{{ allOdds.ratio }}</span> или
                    <span class="big-number">{{ allOdds.perc }}%</span></span
                  >
                </td>

                <td>
                  <div class="prediction-bar prediction-bar--hand">
                    <div :style="`width: ${allOdds.perc}%`"></div>
                  </div>
                </td>
              </tr>
              <tr>
                <td>Шансы банка</td>
                <td>
                  <span class="text-out"
                    ><span class="big-number">{{ potOdds.ratio }}</span> или
                    <span class="big-number">{{ potOdds.perc }}%</span></span
                  >
                </td>
                <td>
                  <div class="prediction-bar prediction-bar--pot">
                    <div :style="`width: ${potOdds.perc}%`"></div>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
          <div class="prediction">
            Рекомендуется
            <br />
            <span :class="[shouldCall ? 'yes' : 'no', 'big-number']">{{
              shouldCall ? "СТАВИТЬ" : "НЕ СТАВИТЬ"
            }}</span>
          </div>
        </div>
      </div>
    </section>
    <footer class="site-footer">
      <div class="container">
        <a
          href="https://forms.gle/4UQpXSpq4guQb2W68"
          target="_blank"
          class="contact-me"
          >Задать вопрос разработчику</a
        >
      </div>
    </footer>
    <svg
      xmlns="http://www.w3.org/2000/svg"
      height="24px"
      viewBox="0 0 24 24"
      width="24px"
      class="below"
      v-if="arrow"
    >
      <path
        d="M20 12l-1.41-1.41L13 16.17V4h-2v12.17l-5.58-5.59L4 12l8 8 8-8z"
      />
    </svg>
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
      excludeCards: [],
      arrow: false,
      isElementInViewport: false,
      myCards: true,
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
          this.excludeCards.length &&
          this.excludeCards.filter(
            (item0) => item0.value === item.value && item0.suit === item.suit
          ).length
        ) {
          temp.opp = true;
        } else {
          temp.opp = false;
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
          ).length === 0 && this.excludeCards.filter(
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
            item.odds = {
              ratio: ratio(item.outs.length, this.unseenDeck.length),
              perc: Math.round(
                (item.outs.length / this.unseenDeck.length) * 100
              ),
            };
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
            item.odds = {
              ratio: ratio(item.outs.length, this.unseenDeck.length),
              perc: Math.round(
                (item.outs.length / this.unseenDeck.length) * 100
              ),
            };
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
    completed: function() {
      let temp0 = [];

      if (this.combAll.length) {
        temp0 = JSON.parse(JSON.stringify(this.combAll));

        temp0 = temp0.filter((item) => item.list.length === 5);

        if (temp0.filter((item) => item.type === "flush").length) {
          temp0 = temp0.filter((item) => item.type === "flush");
        }

        temp0.forEach((item, index) => {
          if (temp0.filter((item1) => item1.type === item.type).length > 1) {
            const maxWeight = Math.max(
              ...temp0
                .filter((item1) => item1.type === item.type)
                .map((item1) => item1.list[4].valueWeight)
            );
            if (item.list[4].valueWeight !== maxWeight) {
              temp0 = temp0.filter((item1, index1) => index1 !== index);
            }
          }
        });
      }

      return temp0;
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
                .map((item1) => item1.list[item1.list.length - 1].valueWeight)
            );
            if (item.list[item.list.length - 1].valueWeight !== maxWeight) {
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
      const call = this.call.length === 0 ? 0 : this.call;
      const pot = this.pot.length === 0 ? 0 : this.pot;

      if (call > 0 && pot > 0) {
        return {
          ratio: ratio(call, pot),
          perc: Math.round((call / (pot + call)) * 100),
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
  watch: {
    shouldCall: function(newVal, oldVal) {
      console.log(newVal)
      if (newVal === null) {
        this.arrow = false;
      } else {
        if (newVal !== oldVal && !this.isElementInViewport) {
          this.arrow = true;
        } else {
          this.arrow = false;
        }
      }
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
      if (this.myCards) {
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
            this.excludeCards = this.excludeCards.filter(
              (item) => !(item.value === card.value && item.suit === card.suit)
            );
            this.concatCards.push(
              this.deck.find(
                (item) => item.suit === card.suit && item.value === card.value
              )
            );
          }
        }
      } else {
        if (
          this.excludeCards.filter(
            (item) => item.value === card.value && item.suit === card.suit
          ).length
        ) {
          this.excludeCards = this.excludeCards.filter(
            (item) => !(item.value === card.value && item.suit === card.suit)
          );
        } else {
          if (
            this.concatCards.filter(
              (item) => !(item.suit === card.suit && item.value === card.value)
            )
          ) {
            this.excludeCards.push(
              this.deck.find(
                (item) => item.suit === card.suit && item.value === card.value
              )
            );
          }
        }
      }
    },
    isNumber: function(evt) {
      evt = evt ? evt : window.event;
      var charCode = evt.which ? evt.which : evt.keyCode;
      if (charCode > 31 && (charCode < 48 || charCode > 57)) {
        evt.preventDefault();
      } else {
        return true;
      }
    },
    visibilityChanged: function(isVisible) {
      this.isElementInViewport = isVisible;
    },
  },
  mounted: function() {
    const handler = () => {
      if (this.isElementInViewport) {
        this.arrow = false;
      }
    };

    addEventListener("scroll", handler, false);
    addEventListener("resize", handler, false);
  },
};
</script>

<style lang="scss">
@import "@/styles/variables";

.mobile-only {
  @media screen and (min-width: 501px) {
    display: none !important;
  }
}

.logo {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.logo img {
  width: 100px;
}

.container {
  max-width: 800px;
  padding: 0 15px;
  margin: 0 auto;
}

.site-header {
  padding: 2rem 0;
  background-color: white;
}

.site-heading {
  font-size: 25px;
  line-height: 1.5;
  text-align: center;
}

.site-heading *:first-child {
  font-weight: bold;
  color: $textColor;
}

.site-heading *:last-child {
  font-size: 0.8em;
  color: $out;
}

.out-section {
  padding: 2rem 0;
}

.deck-wrapper {
  display: flex;
  justify-content: center;
}

.ui-deck {
  display: flex;
  justify-content: center;
  margin: 0 -5px;

  @media screen and (max-width: 900px) {
    flex-wrap: wrap;
    margin: -5px;
  }
}

.ui-deck > li {
  padding: 0 5px;

  @media screen and (max-width: 900px) {
    padding: 5px;
  }
}

.ui-deck__group {
  display: flex;
  flex-direction: column;
  margin: -2px 0;

  @media screen and (max-width: 900px) {
    display: flex;
    flex-wrap: wrap;
    flex-direction: row;
    margin: -2px;
    width: 108px;
  }

  @media screen and (max-width: 500px) {
    width: 88px;
  }
}

.ui-deck__group > li {
  padding: 2px 0;

  @media screen and (max-width: 900px) {
    flex: 0 0 50%;
    max-width: 50%;
    padding: 2px;
  }
}

.card {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 50px;
  height: (50px * 1.390625);
  border: 2px solid transparent;
  border-radius: 5px;
  background-color: white;
  color: $textColor;
  font-family: "Old Standard TT", serif;
  font-size: 24px;
  user-select: none;
  transition: 0.1s linear;
  pointer-events: none;

  @media screen and (max-width: 500px) {
    width: 40px;
    height: 40px;
    font-size: 18px;
  }
}

.card.selectable {
  box-shadow: 0 3px 1px -2px rgba(0, 0, 0, 0.2), 0 2px 2px 0 rgba(0, 0, 0, 0.14),
    0 1px 5px 0 rgba(0, 0, 0, 0.12);
  pointer-events: all;
  cursor: pointer;
}

.card.selectable:hover {
  box-shadow: 0 2px 1px -1px rgba(0, 0, 0, 0.2), 0 1px 1px 0 rgba(0, 0, 0, 0.14),
    0 1px 3px 0 rgba(0, 0, 0, 0.12);
}

.card.selected {
  border-color: $selected;
  box-shadow: none;
}

.card.selected.selectable:hover {
  border-color: darken($selected, 10%);
  box-shadow: none;
}

.card.out {
  border-color: $out;
}

.card.out.selectable:hover {
  border-color: darken($out, 30%);
  box-shadow: none;
}

.card.opp {
  border-color: $redSuit;
  box-shadow: none;
}

.card.opp.selectable:hover {
  border-color: darken($redSuit, 10%);
  box-shadow: none;
}

.card .suit {
  font-size: 1.2em;
}

.red-suit {
  color: $redSuit;
}

.section-heading {
  margin-bottom: 1em;
  font-size: 20px;
  text-align: center;
  font-weight: bold;
  color: $textColor;
}

.section-text {
  margin-bottom: 2.2em;
  font-size: 14px;
  line-height: 1.5;
  color: $textColor;
}

.section-text p {
  margin: 0.8em 0;
}

.section-text p:first-child {
  margin-top: 0;
}

.section-text p:last-child {
  margin-bottom: 0;
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

.message-container > ul {
  width: 500px;
  max-width: 100%;
}

.message {
  display: flex;
  align-items: center;
  justify-content: center;
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
  max-width: 500px;
  margin: 20px auto 0;
  padding: 2em;
  border-radius: 10px;
  background-color: white;
  font-size: 14px;
  line-height: 1.4;
  color: $textColor;
}

* + .outs-summary__section {
  margin-top: 2em;
}

.outs-heading {
  margin-bottom: 0.5em;
  font-weight: bold;
}

.outs-heading--single {
  margin-bottom: 0;
  text-align: center;
}

* + .outs-heading {
  margin-top: 2em;
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
  border-radius: 3px;
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

.draw-table {
  margin: -3px 0;

  @media screen and (max-width: 500px) {
    display: block;
  }
}

.draw-table tbody,
.draw-table tr {
  @media screen and (max-width: 500px) {
    display: block;
  }
}

.draw-table th {
  padding: 3px 6px;
  font-weight: normal;
  text-align: center;

  @media screen and (max-width: 500px) {
    display: none;
  }
}

.draw-table td {
  padding: 3px 6px;
  vertical-align: baseline;

  @media screen and (max-width: 500px) {
    display: block;
    padding: 3px 0;
    text-align: left;
  }
}

.draw-table td:first-child {
  padding-left: 0;
}

.draw-table td:last-child {
  padding-right: 0;
}

.td-outs,
.td-odds {
  text-align: center;
}

.draw-list {
  display: flex;
  margin: 0 -3px;
}

.draw-list li {
  padding: 0 3px;
}

.pot-section {
  padding: 2rem 0;
  background-color: white;
}

.pot-form {
  padding: 2em;
  border: 1px solid $out;
  border-radius: 10px;

  @media screen and (max-width: 500px) {
    padding: 1em;
  }
}

.pot-form__list {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  margin: -10px;
}

.pot-form__cell {
  flex: 0 0 50%;
  max-width: 220px;
  padding: 10px;

  @media screen and (max-width: 500px) {
    flex: 0 0 100%;
  }
}

.text-field-wrapper {
  position: relative;
}

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type="number"] {
  -moz-appearance: textfield;
}

input {
  outline: none;
  box-shadow: none;
}

.text-field {
  display: block;
  height: 50px;
  width: 100%;
  border-bottom: 2px solid darken($background, 20%);
  text-align: center;
  font-size: 30px;
  color: $textColor;
  background-color: darken($background, 1);
}

.text-field:hover {
  border-bottom: 2px solid darken($background, 40%);
}

.text-field:focus {
  border-bottom: 2px solid $out;
}

.text-field-label {
  display: block;
  margin-bottom: 0.5em;
  text-align: center;
  font-size: 14px;
  color: $textColor;
}

.pot-results {
  max-width: 500px;
  margin: 20px auto 0;
  font-size: 16px;
  text-align: center;
}

.prediction-section {
  padding: 2rem 0;
}

.prediction-bar {
  position: relative;
  height: 40px;
  background-color: darken($background, 10%);
}

.prediction-bar > * {
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
}

.prediction-bar--hand > * {
  background-color: $out;
}

.prediction-bar--pot > * {
  background-color: $selected;
}

.prediction-table {
  min-width: 100%;
  margin: -3px 0;

  @media screen and (max-width: 500px) {
    display: block;
  }
}

.prediction-table tbody,
.prediction-table tr {
  @media screen and (max-width: 500px) {
    display: block;
  }
}

.prediction-table th {
  padding: 6px 12px;
  font-weight: normal;
  text-align: center;

  @media screen and (max-width: 500px) {
    display: none;
  }
}

.prediction-table td {
  padding: 6px 12px;
  vertical-align: middle;
  font-size: 16px;
  color: $textColor;

  @media screen and (max-width: 500px) {
    display: block;
    padding: 6px 0;
  }
}

.prediction-table td:first-child {
  padding-left: 0;
}

.prediction-table td:first-child,
.prediction-table td:nth-child(2) {
  width: 0;
  white-space: nowrap;
}

.prediction-table td:last-child {
  padding-right: 0;
}

.prediction {
  margin-top: 2em;
  font-size: 16px;
  text-align: center;
  line-height: 1.5;
  color: $textColor;
}

.prediction .big-number {
  font-weight: bold;
}

.prediction .big-number.yes {
  color: $selected;
}

.prediction .big-number.no {
  color: $redSuit;
}

.deck-legend-wrapper {
  margin-top: 20px;
}

.deck-legend {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  margin: -10px;
}

.deck-legend li {
  padding: 10px;
  font-size: 12px;
  color: $textColor;
}

.deck-legend__box {
  display: inline-block;
  width: 10px;
  height: 10px;
}

.deck-legend__box--hand {
  background-color: $selected;
}

.deck-legend__box--opp {
  background-color: $redSuit;
}

.deck-legend__box--outs {
  background-color: $out;
}

.text-center {
  text-align: center;
}

.formula {
  background-color: rgba(0, 0, 0, 0.08);
  padding: 0.5em;
  font-style: italic;
  font-family: "Old Standard TT", serif;
  font-weight: bold;
  font-size: 1.2em;
}

.site-footer {
  padding: 2rem 0;
  background-color: white;
}

.site-footer .container {
  display: flex;
  justify-content: center;
}

.contact-me {
  color: $out;
  font-size: 12px;
  text-decoration: underline;
}

@keyframes upDown {
  0% {
    transform: translate(-50%, 0);
  }

  50% {
    transform: translate(-50%, -20%);
  }

  100% {
    transform: translate(-50%, 0);
  }
}

.below {
  position: fixed;
  left: 50%;
  bottom: 3vh;
  width: 100px;
  height: auto;
  fill: $selected;
  opacity: 0.3;
  transform: translateX(-50%);
  pointer-events: none;
  animation: upDown 1s linear infinite;
}

.deck-tools {
  margin-bottom: 20px;
}

.switcher {
  display: flex;
  justify-content: center;
}

.switcher > li {
  padding: 0.5em 1em;
  background-color: darken($background, 10%);
  color: $textColor;
  font-size: 14px;
  user-select: none;
  cursor: pointer;
}

.switcher > li:first-child {
  border-top-left-radius: 10px;
  border-bottom-left-radius: 10px;
}

.switcher > li:last-child {
  border-top-right-radius: 10px;
  border-bottom-right-radius: 10px;
}

.switcher > li.my.active {
  background-color: $selected;
  color: white;
}

.switcher > li.opp.active {
  background-color: $redSuit;
  color: white;
}
</style>
