<template>
  <div class="container pl-lg-5">

    <div class="row mt-4 selection-bar">

      <b-input-group class="col-sm-11 pl-0">
        <b-form-select class="col-sm-4" @change="getRaceBonuses(selectedRace)" v-model="selectedRace">
          <template #first>
            <b-form-select-option :value="null" disabled>Select a race</b-form-select-option>
          </template>
          <b-form-select-option v-for="race in races" :value="race.name">{{ race.name }}</b-form-select-option>
        </b-form-select>

        <div class="col-sm-4" v-if="getSubrace(selectedRace)">
          <b-form-select @change="getRaceBonuses(selectedRace)" v-model="selectedSubrace" placeholder="Subrace"
                         list="sub-list">
            <template #first>
              <b-form-select-option :value="null" disabled>Select a Subrace</b-form-select-option>
            </template>
            <b-form-select-option v-for="sub in getSubrace(selectedRace)" :value="sub">{{ sub }}</b-form-select-option>
          </b-form-select>
        </div>

        <b-form-select class="col-sm-4" @change="getClassBonus(selectedClass)" v-model="selectedClass">
          <template #first>
            <b-form-select-option :value="null" disabled>Select a Class</b-form-select-option>
          </template>
          <b-form-select-option v-for="clas in classes" :value="clas.name">{{ clas.name }}</b-form-select-option>
        </b-form-select>
      </b-input-group>
      <b-button class="col-sm-1" @click="reset">Reset</b-button>
    </div>

    <div class="row race-bonus">
      <div v-if="raceBonuses" class="h3 col-sm-12 pl-0 pt-4">
        Race Bonus points: {{ raceBonusPoints }}
      </div>
      <div class="col-sm-12 pl-0" v-if="raceBonuses">
        <b-card-group deck class="col-sm-12">
          <template v-for="stat in stats">
            <template v-for="bonus in raceBonuses.bonusStats">
              <div class="text-center mt-4 col-sm-2 px-0" v-if="bonus === stat.name">
                <b-button @click="raceBonusUp(stat)"
                          :disabled="raceBonusPoints <= 0 || stat.raceBonus >= 1"
                          variant="success" size="lg">
                  <b-icon-arrow-up-short></b-icon-arrow-up-short>
                </b-button>
                <b-card :header="stat.name" :key="stat.id" class="text-center my-2">
                  <b-card-text class="h1">{{ stat.raceBonus }}</b-card-text>
                </b-card>
                <b-button @click="raceBonusDown(stat)" :disabled="stat.raceBonus === 0" variant="danger" size="lg">
                  <b-icon-arrow-down-short></b-icon-arrow-down-short>
                </b-button>
              </div>
            </template>

          </template>
        </b-card-group>
      </div>
    </div>

    <div class="row class-bonus">
      <div v-if="classBonuses" class="h3 col-sm-12 pl-0 pt-4">
        Class Bonus points: {{ classBonusPoints }}
      </div>
      <div v-if="classBonuses" class="col-sm-12 pl-0">
        <b-card-group deck>
          <template v-for="stat in stats">
            <template v-for="bonus in classBonuses">
              <div class="text-center mt-4 col-sm-2 px-0" v-if="bonus === stat.name">
                <b-button @click="classBonusUp(stat)"
                          :disabled="classBonusPoints <= 0 || stat.classBonus >= 2"
                          variant="success" size="lg">
                  <b-icon-arrow-up-short></b-icon-arrow-up-short>
                </b-button>
                <b-card :header="stat.name" :key="stat.id" class="text-center my-2">
                  <b-card-text class="h1">{{ stat.classBonus }}</b-card-text>
                </b-card>
                <b-button @click="classBonusDown(stat)" :disabled="stat.classBonus === 0" variant="danger" size="lg">
                  <b-icon-arrow-down-short></b-icon-arrow-down-short>
                </b-button>
              </div>
            </template>

          </template>
        </b-card-group>
      </div>
    </div>

    <div class="row point-buy">
      <div class="h3 col-sm-12 pl-0 pt-4">PointBuy points: {{ pointBuy }}</div>
      <b-card-group deck class="col-sm-12">
        <div class="text-center mt-4 col-sm-2 px-0" v-for="stat in stats">
          <b-button @click="valueUp(stat)" :disabled="isDisabled(stat)" variant="success" size="lg">
            <b-icon-arrow-up-short></b-icon-arrow-up-short>
          </b-button>
          <b-card :header="stat.name" :key="stat.id" class="text-center my-2">
            <b-card-text class="h1">{{ stat.number + stat.raceBonus + stat.classBonus }}</b-card-text>
          </b-card>
          <b-button @click="valueDown(stat)" :disabled="stat.number === 8" variant="danger" size="lg">
            <b-icon-arrow-down-short></b-icon-arrow-down-short>
          </b-button>
        </div>
      </b-card-group>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  data() {
    return {
      stats: [],
      races: [] as any,
      classes: [] as any,
      pointBuy: 27,
      selectedRace: null,
      selectedSubrace: null,
      selectedClass: null,
      raceBonuses: null,
      classBonuses: null,
      raceBonusPoints: 0,
      classBonusPoints: 2
    }
  },
  async fetch() {
    this.stats = await this.$axios.$get('/stats.json');
    this.races = await this.$axios.$get('/races.json');
    this.classes = await this.$axios.$get('/classes.json');
  },
  methods: {
    valueUp(stat: any): void {
      if (stat.number < 13) {
        stat.number++;
        this.pointBuy--;
      } else if (this.pointBuy >= 2) {
        stat.number++;
        this.pointBuy -= 2;
      }
    },
    valueDown(stat: any): void {
      if (stat.number > 13) {
        stat.number--;
        this.pointBuy += 2;
      } else {
        stat.number--;
        this.pointBuy++;
      }
    },
    isDisabled(stat: any): boolean {
      if (stat.number >= 15 || stat.number >= 13 && this.pointBuy < 2) {
        return true
      } else {
        return this.pointBuy < 1;
      }
    },
    reset(): void {
      this.selectedRace = null;
      this.selectedSubrace = null;
      this.selectedClass = null;
      this.raceBonuses = null;
      this.classBonuses = null;
      this.classBonusPoints = 2;
      this.pointBuy = 27;
      this.stats.forEach((stat: {
        classBonus: number;
        raceBonus: number;
        number: number;
      }) => {
        stat.number = 8;
        stat.raceBonus = 0;
        stat.classBonus = 0;
      });
    },
    getSubrace(name: String) {
      const race = this.races.find((e: { name: String; }) => e.name === name);
      if (race) {
        return race.subrace;
      }
    },
    getRaceBonuses(name: String) {
      // reset the previous bonuses if any are selected
      this.stats.forEach((stat: {
        raceBonus: number;
      }) => {
        stat.raceBonus = 0;
      });
      // find race to get the bonus corresponding the race
      const race = this.races.find((e: { name: String; }) => e.name === name);
      if (race.subrace && this.selectedSubrace) {
        // @ts-ignore
        const bonus = race.bonus[race.subrace.indexOf(this.selectedSubrace)];
        if (bonus) {
          this.raceBonuses = bonus;
        } else {
          this.selectedSubrace = null;
          this.raceBonuses = null;
        }
      } else if (!race.subrace) {
        this.selectedSubrace = null;
        this.raceBonuses = race.bonus[0]
      } else if (this.raceBonuses) {
        this.selectedSubrace = null;
        this.raceBonuses = null;
      }
      if (this.raceBonuses) {
        // @ts-ignore
        this.raceBonusPoints = this.raceBonuses.points;
      }
      return this.raceBonuses;
    },
    raceBonusUp(stat: any) {
      stat.raceBonus++;
      this.raceBonusPoints--;
    },
    raceBonusDown(stat: any) {
      stat.raceBonus--;
      this.raceBonusPoints++;
    },
    getClassBonus(name: String) {
      // reset
      this.classBonusPoints = 2;
      this.stats.forEach((stat: {
        classBonus: number;
      }) => {
        stat.classBonus = 0;
      });
      // search class
      const clas = this.classes.find((e: { name: String; }) => e.name === name);
      this.classBonuses = clas.bonus;
    },
    classBonusUp(stat: any) {
      stat.classBonus++;
      this.classBonusPoints--;
    },
    classBonusDown(stat: any) {
      stat.classBonus--;
      this.classBonusPoints++;
    }
  }
})
</script>
