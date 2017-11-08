<template>
  <div id="app">
    <nav class="navbar is-transparent">
			<div class="navbar-brand">
				<a class="navbar-item" href="https://bulma.io">
					<img src="./assets/logo.png" alt="OK Log" height="28">
				</a>
				<div class="navbar-burger burger" data-target="navbarExampleTransparentExample">
					<span></span>
					<span></span>
					<span></span>
				</div>
			</div>
      <div class="navbar-menu">
        <div class="navbar-item">
          <div class="field is-horizontal">
            <div class="field-label is-normal">
              <label class="label" for="from">From</label>
            </div>
            <div class="field-body">
              <div class="field">
                <div class="control">
                  <div class="select">
                    <select v-model="from" id="from">
                      <option value="-5 minutes">5m</option>
                      <option value="-15 minutes">15m</option>
                      <option value="-1 hour">1h</option>
                      <option value="-12 hours">12h</option>
                      <option value="-1 day">1d</option>
                      <option value="-3 days">3d</option>
                      <option value="-7 days">7d</option>
                      <option>at</option>
                    </select>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="navbar-item">
          <div class="field is-horizontal">
            <div class="field-label is-normal">
              <label class="label" for="from">To</label>
            </div>
            <div class="field-body">
              <div class="field">
                <div class="control">
                  <div class="select">
                    <select v-model="to" id="from">
                      <option>now</option>
                      <option>at</option>
                    </select>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="navbar-item">
          <div class="field  has-addons has-addons-right">
            <p class="control">
              <input v-model="filterExpr" id="filter" class="input" type="text" placeholder="jq filter expression">
            </p>
            <p class="control">
              <button @click="getLines" class="button is-primary">Go</button>
            </p>
          </div>
        </div>
      </div>
    </nav>
    <section class="section">
      <div class="columns">
        <div class="column is-2">
          <nav class="panel">
            <p class="panel-heading">OK Log</p>
            <div class="panel-block">
              <p class="control has-icons-left">
                <input v-model="hostname" class="input is-small" type="text" placeholder="hostname">
              </p>
            </div>
            <a class="panel-block is-active">
              <span class="panel-icon">
                <i class="fa fa-book"></i>
              </span>
              bulma
            </a>
            <div class="panel-block">
              <button class="button is-link is-outlined is-fullwidth">
                reset all filters
              </button>
            </div>
          </nav>
        </div>
        <div class="column">
          <ul>
            <li v-for="l in lines">{{ l }}</li>
          </ul>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import axios from 'axios';
import jq from 'jq-web';

const moment = require('moment');
require('moment-parseplus');

export default {
  name: 'app',
  data() {
    return {
      hostname: '',
      filterExpr: '',
      from: '-5 minutes',
      to: 'now',
      lines: [],
    };
  },
  computed: {
    toTimestamp() {
      if (this.to === 'now') {
        return moment().format();
      }

      return this.to;
    },
    fromTimestamp() {
      if (this.from === 'at') {
        return this.from;
      }

      return moment(this.from).format();
    },
  },
  methods: {
    getLines() {
      axios.get(`${this.hostname}/store/query`, {
        params: {
          from: this.fromTimestamp,
          to: this.toTimestamp,
        },
      }).then((res) => {
        this.lines = res.data.split('\n').map(l => l.slice(27)).map((l) => {
          try {
            return JSON.parse(l);
          } catch (e) {
            return null;
          }
        }).filter(l => l !== null);
        if (this.filterExpr !== '') {
          this.lines = jq(this.lines, this.filterExpr);
        }
      });
    },
  },
};
</script>

<style lang="scss">
  @import "./assets/css/application";

  #filter {
    width: 40rem;
  }
</style>
