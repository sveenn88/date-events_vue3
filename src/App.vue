<template>
  <div class="main">
    <DateEvents :events="events" />
  </div>
</template>

<script>
import DateEvents from "./components/DateEvents.vue";

export default {
  name: "App",
  components: {
    DateEvents,
  },
  data() {
    return {
      events: [
        {
          id: 1,
          name: "Событе какое-то sdgdgsdgsdg",
          date: 1633027358,
          type: 0,
        },
        {
          id: 2,
          name: "Событе какое-то sdgdgsdgsdg",
          date: 1632940858,
          type: 1,
        },
        {
          id: 3,
          name: "Событе какое-то eweffwef",
          date: 1632940458,
          type: 2,
        },
        {
          id: 4,
          name: "Событе какое-то wefwe tyhtyhtyht",
          date: 1632767858,
          type: 1,
        },
        {
          id: 5,
          name: "Событе какое-то wef efergergergerg",
          date: 1632746258,
          type: 0,
        },
        {
          id: 6,
          name: "Событе какое-то qwgbrtb кепкепкеп grgrtg",
          date: 1632577538,
          type: 2,
        },
        {
          id: 7,
          name: "Событе какое-то bmuol gererverv rverger",
          date: 1632595538,
          type: 1,
        },
        {
          id: 8,
          name: "Событе какое-то asd vrtgrh неркеркенрен",
          date: 1632599138,
          type: 0,
        },
        {
          id: 9,
          name: "Постояное событие",
          date: new Date().getTime() / 1000, // чтоб всегда было видно какие-то события
          type: 0,
        },
        {
          id: 10,
          name: "Постояное событие",
          date: new Date().getTime() / 1000 - (24 * 60 * 60),
          type: 1,
        },
        {
          id: 11,
          name: "Постояное событие",
          date: new Date().getTime() / 1000 + (24 * 60 * 60),
          type: 2,
        },
        {
          id: 12,
          name: "Постояное событие",
          date: new Date().getTime() / 1000 + (2 * 60 * 60),
          type: 1,
        },
      ],
    };
  },
};
</script>

<style>
</style>
