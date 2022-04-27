<template>
  <q-page class="flex column">
    <div class="col q-pt-lg q-px-md text-center">
      <div class="q-pa-md absolute-center">
        <template v-if="!stage">
          <q-input
            rounded
            standout
            suffix="minutes"
            type="number"
            v-model="max_minutes"
            v-if="!stage"
            label="Target time away *"
            lazy-rules
            :rules="[
              (val) =>
                (val !== null && val !== '') || 'Enter number of minutes',
              (val) => (val > 0 && val < 240) || 'Must be less than 4 hours',
            ]"
          />

          <br />

          <q-input
            rounded
            standout
            type="number"
            suffix="reps"
            v-model="num_reps"
            v-if="!stage"
            label="Number of reps *"
            lazy-rules
            :rules="[
              (val) => (val !== null && val !== '') || 'Enter number of reps',
              (val) => (val > 0 && val < 15) || 'Must be less than 15',
            ]"
          />

          <br />

          <div>
            <q-btn
              v-if="!stage"
              @click="onSubmit"
              label="Submit"
              type="submit"
              color="primary"
            />
          </div>
        </template>
      </div>
      <div class="q-pa-md absolute-center">
        <template v-if="stage">
          <div class="col text-center">
            <div class="text-h4 text-weight-light">{{ stage }}</div>
          </div>

          <br />

          <q-circular-progress
            :min="0"
            :max="max_seconds"
            show-value
            class="text-light-blue q-ma-md"
            :value="time_away"
            size="200px"
            color="light-blue"
          />

          <br />
          <br />

          <div>
            <q-btn
              v-if="!paused"
              @click="onPause"
              label="Pause"
              type="submit"
              color="primary"
              class="q-ml-sm"
            />
            <q-btn
              v-if="paused"
              @click="onResume"
              label="Resume"
              type="submit"
              color="primary"
              class="q-ml-sm"
            />

            <q-btn
              @click="onReset"
              label="Reset"
              type="reset"
              color="primary"
              flat
              class="q-ml-sm"
            />
          </div>

          <br />
          <br />

          <div class="col text-center">
            <div class="text-h5 text-weight-light">
              Current interval: {{ current }}
              <span v-if="current">seconds</span>
            </div>
          </div>

          <br />
          <div class="col text-center">
            <div class="text-h5 text-weight-light">
              Upcoming intervals:
              <br />
              {{ upcoming }}
            </div>
          </div>

          <br />

          <div class="col text-center">
            <div class="text-h5 text-weight-light">
              Finished intervals:
              <br />{{ done }}
            </div>
          </div>
        </template>
      </div>
    </div>
  </q-page>
</template>

<script>
import { useQuasar } from "quasar";
import { ref } from "vue";

export default {
  data() {
    return {
      max_minutes: ref(1),
      num_reps: ref(8),

      time_away: null,
      max_seconds: ref(5),
      stage: null,
      paused: ref(false),
      intervals: null,
      done: null,
      upcoming: null,
      current: null,
    };
  },
  methods: {
    // When submit button is clicked
    onSubmit() {
      this.stage = "Stand up and leave";
      var audio = new Audio("standup.mp3");
      audio.play();

      this.max_seconds = this.max_minutes * 60;

      // Calculate time intervals
      const standup = 10;
      var times = [standup, 5, 30, standup, 15, 30]; // Standup, away, relax. Standup, away, relax.
      this.intervals = [5, 15];
      var num_unique_reps = this.num_reps / 2;
      var step = Math.floor(this.max_seconds / num_unique_reps);
      for (let i = 0; i < num_unique_reps; i++) {
        var target = step * (i + 1); // Average target for 2 reps
        var minimum = Math.floor(step / 4);
        var maximum = Math.floor(step / 2);
        var random =
          Math.floor(Math.random() * (maximum - minimum + 1)) + minimum;
        times.push(standup);
        times.push(target + random);
        this.intervals.push(target + random);
        var relax = Math.floor(Math.random() * (45 - 30 + 1)) + 30;
        times.push(relax);
        times.push(standup);
        times.push(target - random);
        this.intervals.push(target - random);
        var relax = Math.floor(Math.random() * (45 - 30 + 1)) + 30;
        times.push(relax);
      }

      // Set up next circular progress
      this.time_away = this.max_seconds;
      this.upcoming = this.intervals.slice(1, -1).join(", ");
      this.current = this.intervals[0];

      // List of time intervals to leave
      var times_counter = 0;
      var interval_counter = 0;

      this.time_away = times[0];
      this.max_seconds = this.time_away;

      // Begin the loop

      // Timer loop
      var interval = setInterval(() => {
        if (!this.paused) {
          this.time_away--;
          // Play sounds if you are away
          if (this.stage == "Wait outside") {
            if (this.time_away == 10) {
              var audio = new Audio("ten.mp3");
              audio.play();
            }
            if (this.time_away == 5) {
              var audio = new Audio("five.mp3");
              audio.play();
            }
            if (this.time_away == 4) {
              var audio = new Audio("four.mp3");
              audio.play();
            }
            if (this.time_away == 3) {
              var audio = new Audio("three.mp3");
              audio.play();
            }
            if (this.time_away == 2) {
              var audio = new Audio("two.mp3");
              audio.play();
            }
            if (this.time_away == 1) {
              var audio = new Audio("one.mp3");
              audio.play();
            }
            if (this.time_away == 0) {
              var audio = new Audio("return.mp3");
              audio.play();
            }
          }

          // If time is up on the counter, move to next item in time array
          if (this.time_away == -1) {
            times_counter++;
            // Finishing stand up, starting to wait
            if (this.stage == "Stand up and leave") {
              this.stage = "Wait outside";

              var audio = new Audio("begin.mp3");
              audio.play();
            } // Finishing waiting, starting to return
            else if (this.stage == "Wait outside") {
              this.stage = "Return and relax";

              interval_counter++;
              this.current = null;
              this.done = this.intervals.slice(0, interval_counter).join(", ");
              this.upcoming = this.intervals
                .slice(interval_counter, -1)
                .join(", ");
            } // Finishing return and relax, starting to stand up
            else if (this.stage == "Return and relax") {
              this.current = this.intervals[interval_counter];
              this.upcoming = this.intervals
                .slice(interval_counter + 1, -1)
                .join(", ");
              this.stage = "Stand up and leave";

              var audio = new Audio("standup.mp3");
              audio.play();
            }
            if (times_counter == times.length) {
              clearInterval(interval);
            }
            this.time_away = times[times_counter];

            this.max_seconds = this.time_away;
          }
        }
      }, 1000);
    },

    onPause() {
      this.paused = true;
    },
    onResume() {
      this.paused = false;
    },
    onReset() {
      location.reload();
    },
  },
};
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to top, #74ebd5, #acb6e5)
.timer-form
</style>
