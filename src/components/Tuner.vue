<template>
  <section class="tuner" :class="{'out-of-tune': direction !== 'OK!', 'in-tune': direction === 'OK!'}">
    <div>{{ note }}</div>
    <div>
      {{ frequency.toFixed(2) }}Hz
    </div>
    <div>{{ direction }}</div>
  </section>
</template>

<script>
export default {
  name: 'Tuner',

  props: {
    frequency: Number,
  },

  data() {
    return {
      direction: null,
    };
  },

  methods: {
    notesRange() {
      const notesRange = new Map();

      notesRange.set('a', 27.50);
      notesRange.set('a#', 29.14);
      notesRange.set('b', 30.87);
      notesRange.set('c', 16.35);
      notesRange.set('c#', 17.32);
      notesRange.set('d', 18.35);
      notesRange.set('d#', 19.45);
      notesRange.set('e', 20.60);
      notesRange.set('f', 21.83);
      notesRange.set('f#', 23.12);
      notesRange.set('g', 24.50);
      notesRange.set('g#', 25.96);

      return notesRange;
    },

    calculateNote() {
      const notesRange = this.notesRange();

      let diff = 1000;
      let detectedKey = null;
      let direction = '';

      for (const [key, freq] of notesRange) {
        const octaves = this.calculateOctaves(freq);

        for (let i = 0; i < octaves.length; i++) {
          const currentDiff = this.frequency - octaves[i];
          const unsignedDiff = this.unsignedDiff(currentDiff);

          if (unsignedDiff < diff) {
            diff = unsignedDiff;
            detectedKey = key;
            direction = this.getDirection(octaves[i], currentDiff)
          }
        }
      }

      this.direction = direction;

      return detectedKey ?? '!';
    },

    calculateOctaves(frequency) {
      const intervals = [];
      let prev = frequency;
      intervals.push(prev);

      for (let i = 0; i < 8; i++) {
        const next = prev * 2;

        intervals.push(next);

        prev = next;
      }

      return intervals;
    },

    unsignedDiff(diff) {
      if (diff < 0) {
        return diff * (-1);
      }

      return diff;
    },

    getDirection(octave, diff) {
      const margin = octave * 0.01;
      const positiveMargin = octave + margin;
      const negativeMargin = octave - margin;

      const frequencyIsInsideMargins = this.frequency >= negativeMargin && this.frequency <= positiveMargin;

      if (frequencyIsInsideMargins) {
        return 'OK!';
      }

      if (diff < 0) {
        return 'UP';
      }

      return 'DOWN';
    },
  },

  computed: {
    note() {
      return String(this.calculateNote()).toUpperCase();
    }
  }
}
</script>

<style scoped>
.tuner {
  display: flex;
  flex-direction: column;
  row-gap: 20px;
  align-items: center;
  margin: 4em;
  font-family: sans-serif;
  font-size: 4em;
}

.out-of-tune {
  color: red;
}

.in-tune {
  color: green;
}
</style>