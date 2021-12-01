<template>
  <div class="container pl-lg-5 mh-100">

    <div class="row pt-4 selection-bar">

      <b-input-group class="col-sm-11">
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
      <div class="col-sm-1">
        <b-button @click="reset">Reset</b-button>
      </div>
    </div>

    <div class="pt-1">
      <b-table responsive :items="stats" :fields="fields">

        <template #head(name)="data">
          <span>Attribute</span>
        </template>

        <template #cell(name)="data">
          <span class="h3">{{ data.item.name }}</span>
        </template>

        <template #head(number)="data">
          PointBuy
        </template>

        <template #cell(number)="data">
          <span class="h1">{{ data.item.number }}</span>
        </template>

        <template #head(numberBtn)="data">
          <span>({{ pointBuy }})</span>
        </template>

        <template #cell(numberBtn)="data">
          <div class="d-inline-grid text-center">
            <b-button @click="valueUp(data.item)"
                      :disabled="isDisabled(data.item)"
                      variant="success" size="sm">
              <b-icon-arrow-up-short></b-icon-arrow-up-short>
            </b-button>
            <b-button @click="valueDown(data.item)"
                      :disabled="data.item.number === 8"
                      variant="danger" size="sm">
              <b-icon-arrow-down-short></b-icon-arrow-down-short>
            </b-button>
          </div>
        </template>

        <template #cell(raceBonus)="data">
          <div v-if="selectedRace && raceBonuses">
            <div v-if="raceBonuses.bonusStats.includes(data.item.name)" class="d-inline-grid text-center">
              <span class="h1">{{ data.item.raceBonus }}</span>
            </div>
          </div>
        </template>

        <template #head(raceBonusBtn)="data">
          <div v-if="selectedRace && raceBonuses">
            ({{ raceBonusPoints }})
          </div>
          <div v-else></div>
        </template>

        <template #cell(raceBonusBtn)="data">
          <div v-if="selectedRace && raceBonuses">
            <div v-if="raceBonuses.bonusStats.includes(data.item.name)" class="d-inline-grid text-center">
              <b-button @click="raceBonusUp(data.item)"
                        :disabled="raceBonusPoints <= 0 || data.item.raceBonus >= 1"
                        variant="success" size="sm">
                <b-icon-arrow-up-short></b-icon-arrow-up-short>
              </b-button>
              <b-button @click="raceBonusDown(data.item)"
                        :disabled="data.item.raceBonus === 0"
                        variant="danger" size="sm">
                <b-icon-arrow-down-short></b-icon-arrow-down-short>
              </b-button>
            </div>
          </div>
        </template>

        <template #head(classBonus)="data">
          <div v-if="selectedRace && classBonuses">
            Class Bonus
          </div>
        </template>

        <template #cell(classBonus)="data">
          <div v-if="selectedClass && classBonuses">
            <div v-if="classBonuses.includes(data.item.name)" class="d-inline-grid text-center">
              <span class="h1">{{ data.item.classBonus }}</span>
            </div>
          </div>
        </template>

        <template #head(classBonusBtn)="data">
          <div v-if="selectedRace && classBonuses">
            ({{ classBonusPoints }})
          </div>
          <div v-else></div>
        </template>

        <template #cell(classBonusBtn)="data">
          <div v-if="selectedClass && classBonuses">
            <div v-if="classBonuses.includes(data.item.name)" class="d-inline-grid text-center">
              <b-button @click="classBonusUp(data.item)"
                        :disabled="classBonusPoints <= 0 || data.item.classBonus >= 2"
                        variant="success" size="sm">
                <b-icon-arrow-up-short></b-icon-arrow-up-short>
              </b-button>
              <b-button @click="classBonusDown(data.item)"
                        :disabled="data.item.classBonus === 0"
                        variant="danger" size="sm">
                <b-icon-arrow-down-short></b-icon-arrow-down-short>
              </b-button>
            </div>
          </div>
        </template>

        <template #cell(totalScore)="data">
          <span class="h1 text-center">{{ getTotal(data.item) }}</span>
        </template>

        <template #cell(modifier)="data">
          <span class="h3 text-center">{{ getModifier(data.item) }}</span>
        </template>

      </b-table>
    </div>

  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  data() {
    return {
      fields: [{
        key: "name",
        thClass: "align-middle text-left",
        tdClass: "align-middle text-left"
      }, {
        key: "number",
        thClass: "align-middle text-right",
        tdClass: "align-middle text-right"
      }, {
        key: "numberBtn",
        thClass: "align-middle text-left",
        tdClass: "align-middle text-left"
      }, {
        key: "raceBonus",
        thClass: "align-middle text-right",
        tdClass: "align-middle text-right"
      }, {
        key: "raceBonusBtn",
        thClass: "align-middle text-left",
        tdClass: "align-middle text-left"
      }, {
        key: "classBonus",
        thClass: "align-middle text-right",
        tdClass: "align-middle text-right"
      }, {
        key: "classBonusBtn",
        thClass: "align-middle text-left",
        tdClass: "align-middle text-left"
      }, {
        key: "totalScore",
        thClass: "align-middle text-center",
        tdClass: "align-middle text-center"
      }, {
        key: "modifier",
        thClass: "align-middle text-center",
        tdClass: "align-middle text-center"
      }],
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
    },
    getTotal(stat: any) {
      if (stat) {
        return (stat.number + stat.raceBonus + stat.classBonus);
      }
      return 8;
    },
    getModifier(stat: any) {
      if (stat) {
        const total = this.getTotal(stat);
        if (total <= 9) {
          return "-1";
        } else if (total <= 11) {
          return "0";
        } else if (total <= 13) {
          return "+1";
        } else if (total <= 15) {
          return "+2";
        } else if (total <= 17) {
          return "+3";
        } else if (total === 18) {
          return "+4";
        }
      }
      return -1;
    }
  }
})
</script>
<style>
.d-inline-grid {
  display: inline-grid;
}
</style>
