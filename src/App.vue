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
              <div class="field has-addons">
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
                <div v-if="from == 'at'" class="control">
                  <input class="input" placeholder="iso8601 timestamp" v-model="at">
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
              <div class="field has-addons">
                <div class="control">
                  <div class="select">
                    <select v-model="to" id="from">
                      <option>now</option>
                      <option>at</option>
                    </select>
                  </div>
                </div>
                <div v-if="to == 'at'" class="control">
                  <input class="input" placeholder="iso8601 timestamp" v-model="to_at">
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
          <nav class="panel" id="hosts">
            <p class="panel-heading">OK Log Hosts</p>
            <div class="panel-block">
              <div class="field has-addons">
                <p class="control is-expanded">
                  <input v-model="newHostname" class="input is-small" type="text" placeholder="hostname">
                </p>
                <p class="control">
                  <button @click="createHost" class="button is-primary is-small">Create</button>
                </p>
              </div>
            </div>
            <p class="panel-tabs">
              <a v-for="h in hosts" @click="activeHost = h.hostname" v-bind:class="{'is-active': h.hostname === activeHost}">{{ h.hostname }}</a>
            </p>
            <div class="panel-block">
              <button @click="deleteHost(activeHost)" class="button is-link is-outlined is-fullwidth is-small">delete host</button>
            </div>
            <a class="panel-block is-active">
              <span class="panel-icon">
                <i class="fa fa-book"></i>
              </span>
              bulma
            </a>
          </nav>
        </div>
        <div class="column">
          <div class="notification is-danger" v-if="error"><button @click="error = null" class="delete"></button>{{ error }}</div>
          <table class="table is-striped">
            <thead>
              <tr>
                <th>At</th>
                <th>Data</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="l in lines">
                <td>{{ formatUnix(l.at) }}</td>
                <td><pre>{{ l }}</pre></td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
/* eslint no-underscore-dangle: ["error", { "allow": ["_json", "_at"] }] */
import axios from 'axios';
import jq from 'jq-web';
import Dexie from 'dexie';
import { decodeTime } from 'ulid';

const moment = require('moment');
require('moment-parseplus');

export default {
  name: 'app',
  data() {
    return {
      error: null,
      hosts: [],
      newHostname: null,
      activeHost: null,
      filterExpr: '',
      from: '-5 minutes',
      at: null,
      to: 'now',
      to_at: null,
      lines: [],

      db: null,
    };
  },
  computed: {
    toTimestamp() {
      if (this.to === 'now') {
        return moment().format();
      }

      return this.to_at;
    },
    fromTimestamp() {
      if (this.from === 'at') {
        return this.at;
      }

      return moment(this.from).format();
    },
  },
  methods: {
    formatUnix(t) {
      return moment(t).format();
    },
    getLines() {
      this.error = null;

      axios.get(`${this.activeHost}/store/query`, {
        params: {
          from: this.fromTimestamp,
          to: this.toTimestamp,
        },
      }).then((res) => {
        this.lines = res.data.split('\n').map(l => [l.substr(0, l.indexOf(' ')), l.substr(l.indexOf(' ') + 1)]).map((l) => {
          if (l[0] === '') {
            return null;
          }

          try {
            const d = JSON.parse(l[1]);
            d._json = true;
            d._at = decodeTime(l[0]);
            return d;
          } catch (e) {
            return {
              _at: decodeTime(l[0]),
              data: l[1],
              _json: false,
            };
          }
        }).filter(l => !!l);

        if (this.filterExpr !== '' && this.lines.length > 0) {
          try {
            this.lines = jq(this.lines.filter(l => l._json), this.filterExpr);
            if (!Array.isArray(this.lines)) {
              this.lines = [this.lines];
            }
          } catch (e) {
            this.error = 'An error occurred parsing the json. Make sure your jq filter is formatted properly!';
          }
        }
      }).catch((err) => {
        this.error = `An error occurred getting the data:\n${err.message}`;
      });
    },
    createHost() {
      this.db.hosts.put({
        hostname: this.newHostname,
        filters: [],
      }).then(key => this.db.hosts.get(key)).then((h) => {
        this.hosts.push(h);
        this.activeHost = h.hostname;
      }).catch((err) => {
        this.error = err;
      });
    },
    deleteHost(h) {
      this.db.hosts.delete(h).then(() => {
        this.hosts.splice(
          this.hosts.findIndex(host => host.hostname === h),
          1,
        );

        if (this.hosts.length > 0) {
          this.activeHost = this.hosts[0].hostname;
        }
      });
    },
  },
  mounted() {
    this.db = new Dexie('oklog.db');
    this.db.version(1).stores({
      hosts: 'hostname,filters',
    });
    this.db.hosts.each((h) => {
      this.hosts.push(h);
      this.activeHost = h.hostname;
    });
  },
};
</script>

<style lang="scss">
  @import "./assets/css/application";

  #filter {
    width: 40rem;
  }
</style>
