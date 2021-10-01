<template>
  <div class="calendar-events">
    <div class="calendar-events__nav">
      <div class="nav">
        <div class="nav__prev" @click="setPrevMonth"></div>
        <div class="nav__current">{{ viewValue }}</div>
        <div class="nav__next" @click="setNextMonth"></div>
      </div>
    </div>
    <div class="calendar-events__content">
      <div class="content">
        <div class="content__days">
          <div :class="['content__day', {weekend: index >= 5}]" v-for="(day, index) in weekDays" :key="day.id">
            {{ day }}
          </div>
        </div>
        <div class="content__cells">
          <div
            :class="['content__cell', ...date.class]"
            v-for="date in daysInMonth"
            :key="date"
          >
            <div class="count">{{ date.day }}</div>
            <div class="event-box">
              <div
                v-for="e in date.events"
                :key="e"
                :class="['event', typeEvent[e.type]]"
              >
                <span class="event__time">{{ e.date.format("hh:mm") }}</span>
                <span class="event__name">{{ e.name }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import * as moment from "moment";
import "moment/locale/ru";
export default {
  name: "DateEvents",
  props: {
    events: {
      type: Array,
    },
  },
  mounted() {
    moment.locale("ru");
  },
  computed: {
    // объект для быстрого доступа
    mEvents() {
      let mEvents = {};
      this.events.forEach((e) => {
        e.date = moment(e.date * 1000);
        let key = e.date.format("YYYY-MM-DD");
        mEvents[key] ? mEvents[key].push(e) : (mEvents[key] = [e]);
      });
      return mEvents;
    },
    // месяц || месяц + год
    viewValue() {
      return this.cDate.get("year") === moment().get("year")
        ? this.cDate.format("MMMM")
        : this.cDate.format("MMMM YYYY");
    },
    // Названия дней недели
    weekDays() {
      return [
        ...moment.weekdays().splice(1, 6),
        ...moment.weekdays().splice(0, 1),
      ];
    },
    // массив для календаря
    daysInMonth() {
      return [
        ...this.getDaysOfNextMonth(),
        ...this.getDaysOfCurrentMonth(),
        ...this.getDaysOfPrevMonth(),
      ].reverse();
    },
  },
  data() {
    return {
      cDate: moment().startOf("month"), // текущий месяц с первого числа (для навигации)
      currentDate: moment(), // текущая дата (для стилей)
      typeEvent: { 0: "--important--", 1: "--job--", 2: "--hobby--" }, // типы событий (для стилей)
    };
  },
  methods: {
    // получить событя для даты
    getEventsForDay(fDate) {
      return this.mEvents[fDate] || [];
    },
    // дни текущего месяца
    getDaysOfCurrentMonth() {
      let arrDays = [];
      const countDays = this.cDate.daysInMonth();
      for (let i = countDays; i > 0; i--) {
        let classes = [];
        let currentDate = this.cDate.clone().add(i - 1, "day");

        // до текущей даты
        if (this.isBefore(currentDate, this.currentDate))
          classes.push("--prev--");

        // выходной
        if (this.isWeekend(currentDate)) classes.push("--weekend--");

        // текущая дата
        if (this.isSame(currentDate, this.currentDate))
          classes.push("--current--");

        arrDays.push({
          day: i,
          class: classes || "",
          events: this.getEventsForDay(currentDate.format("YYYY-MM-DD")),
        });
      }
      return arrDays;
    },
    // дни следующего месяца
    getDaysOfNextMonth() {
      let arrDays = [];
      const nextMoment = this.cDate.clone().add(1, "month");
      for (
        let index =
          nextMoment.day() !== 1
            ? nextMoment.day() !== 0
              ? 7 - nextMoment.day()
              : 0
            : -1;
        index > -1;
        index--
      ) {
        const classes = [];
        let nextDay = nextMoment.clone().add(index, "day");
        // до текущей даты
        if (this.isBefore(nextDay, this.currentDate)) classes.push("--prev--");

        // выходной
        if (this.isWeekend(nextDay)) classes.push("--weekend--");

        // текущая дата
        if (this.isSame(nextDay, this.currentDate)) classes.push("--current--");

        arrDays.push({
          day: index + 1,
          class: classes,
          events: this.getEventsForDay(nextDay.format("YYYY-MM-DD")),
        });
      }
      return arrDays;
    },
    // дни предыдущего месяца
    getDaysOfPrevMonth() {
      let arrDays = [];
      const prevMoment = this.cDate.clone().add(-1, "month");
      const countPrevDays = prevMoment.daysInMonth();
      for (
        let index = 0;
        index < (this.cDate.day() - 1 > -1 ? this.cDate.day() - 1 : 6);
        index++
      ) {
        let classes = [];
        let prevDate = prevMoment.clone().add(countPrevDays - index - 1, "day");

        // до текущей даты
        if (this.isBefore(prevDate, this.currentDate)) classes.push("--prev--");

        // выходной?
        if (this.isWeekend(prevDate)) classes.push("--weekend--");

        // текущая дата
        if (this.isSame(prevDate, this.currentDate))
          classes.push("--current--");

        arrDays.push({
          day: countPrevDays - index,
          class: classes,
          events: this.getEventsForDay(prevDate.format("YYYY-MM-DD")),
        });
      }
      return arrDays;
    },
    // установить на предыдущий месяц
    setPrevMonth() {
      this.cDate = this.cDate.clone().set("month", this.cDate.get("month") - 1);
    },
    // установить на следующий месяц
    setNextMonth() {
      this.cDate = this.cDate.clone().set("month", this.cDate.get("month") + 1);
    },
    // это предыдущий?
    isBefore(a, b) {
      return moment(a.format("YYYY-MM-DD"), "YYYY-MM-DD").isBefore(
        b.format("YYYY-MM-DD")
      );
    },
    // это тоже самое?
    isSame(a, b) {
      return moment(a.format("YYYY-MM-DD"), "YYYY-MM-DD").isSame(
        b.format("YYYY-MM-DD")
      );
    },
    // это выходной?
    isWeekend(d) {
      return d.day() === 0 || d.day() === 6;
    },
  },
};
</script>

<style lang="scss" scoped>
.calendar-events {
  font-family: "Montserrat";
  width: auto;
  max-width: 1200px;
  padding: 20px;
  border: 1px solid rgb(228, 225, 225);
  border-radius: 25px;
  box-shadow: 0 3px 1px -2px rgba(0, 0, 0, 0.2), 0 2px 2px 0 rgba(0, 0, 0, 0.14),
    0 1px 5px 0 rgba(0, 0, 0, 0.12);
  margin: 15px;
  &__nav {
    .nav {
      display: flex;
      justify-content: flex-start;
      align-items: center;
      min-height: 40px;
      &__current {
        font-size: 16px;
        font-weight: 600;
        text-transform: uppercase;
        margin: 0 10px;
      }
      &__next,
      &__prev {
        width: 15px;
        height: 15px;
        background-image: url("../assets/img/calendar-events/arrow.svg");
        background-position: center;
        background-repeat: no-repeat;
        background-size: cover;
        cursor: pointer;
      }
      &__prev {
        transform: rotate(180deg);
      }
    }
  }
  &__content {
    .content {
      width: calc(7 * (10vw + 4px));
      max-width: calc(7 * 154px);
      min-width: calc(7 * 81px);
      &__days {
        display: flex;
        flex-wrap: nowrap;
      }
      &__day {
        display: flex;
        flex-shrink: 0;
        justify-content: flex-end;
        align-items: center;
        max-width: 150px;
        min-width: 77px;
        width: 10vw;
        margin: 2px;
        padding: 10px 5px;
        border-radius: 5px;
        font-size: 12px;
        font-weight: 600;
        text-transform: uppercase;
        @media screen and (max-width: 1120px) {
          font-size: 1.05vw;
        }
        @media screen and (max-width: 768px) {
          font-size: 8px;
        }
      }
      &__day.weekend {
        color: #d90aec;
      }
      &__cells {
        display: flex;
        flex-wrap: wrap;
      }
      &__cell {
        max-width: 150px;
        min-width: 77px;
        width: 10vw;
        max-height: 150px;
        min-height: 77px;
        height: 10vw;
        border: 1px solid #d9d8d8;
        margin: 2px;
        border-radius: 5px;
        flex-shrink: 0;
        box-shadow: 0 3px 1px -2px rgba(0, 0, 0, 0.2),
          0 2px 2px 0 rgba(0, 0, 0, 0.14), 0 1px 5px 0 rgba(0, 0, 0, 0.12);
        .count {
          display: flex;
          justify-content: flex-end;
          width: 100%;
          padding: 5px;
          font-size: 15px;
          font-weight: 700;
          color: black;
        }
        .event-box {
          display: flex;
          flex-wrap: wrap;
          padding: 5px;
          .event {
            display: flex;
            flex-wrap: nowrap;
            width: 100%;
            padding: 5px;
            margin-bottom: 5px;
            background-color: white;
            border-radius: 5px;
            &__time {
              margin-right: 3px;
              min-width: 35px;
            }
            &__time,
            &__name {
              font-size: 12px;
              font-weight: 500;
              white-space: nowrap;
              overflow: hidden;
              text-overflow: ellipsis;
            }
          }
          .event:hover {
            width: auto;
            z-index: 1;
            box-shadow: 0 3px 1px -2px rgba(0, 0, 0, 0.2),
              0 2px 2px 0 rgba(0, 0, 0, 0.14), 0 1px 5px 0 rgba(0, 0, 0, 0.12);
            .event__name {
              overflow: unset;
            }
          }
          .event.--hobby-- {
            background-color: rgb(164, 212, 164);
            color: green;
          }
          .event.--job-- {
            background-color: rgb(221, 221, 131);
            color: rgb(180, 180, 7);
          }
          .event.--important-- {
            background-color: rgb(228, 166, 166);
            color: rgb(160, 12, 12);
          }
        }
      }
      &__cell.--prev-- {
        background-color: #d9d8d8;
        box-shadow: none;
        .count {
          color: #767676;
        }
      }
      &__cell.--weekend-- {
        .count {
          color: #d90aec;
        }
      }
      &__cell.--current-- {
        background-color: white;
        border-color: #333;
        .count {
          color: #18bb3b;
        }
      }
    }
  }
}
</style>