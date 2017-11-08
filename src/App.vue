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
                    <select id="from">
                      <option>5m</option>
                      <option>15m</option>
                      <option>1h</option>
                      <option>12h</option>
                      <option>1d</option>
                      <option>3d</option>
                      <option>7d</option>
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
                    <select id="from">
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

export default {
  name: 'app',
  data() {
    return {
      hostname: '',
      filterExpr: '',
      lines: [],
    };
  },
  methods: {
    getLines() {
      axios.get(`${this.hostname}/store/query`, {
        params: {
          from: '2017-11-07T00:00:00-06:00',
          to: '2017-11-08T00:30:00-06:00',
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
