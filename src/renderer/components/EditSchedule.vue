<template>
  <b-container>
    <b-row>
      <b-col>
        <h3>Properties</h3>
      </b-col>
    </b-row>

    <b-row>
      <b-col>
        <p>
          <a href="#" style="color: black" v-on:click.prevent="deleteObject">
            <font-awesome-icon icon="trash"></font-awesome-icon>
          </a>
          type: {{ editObject.type }}
        </p>
      </b-col>
    </b-row>

    <b-row v-if="editObject && editObject.type !== 'group'">
      <b-col>
        <b-form-checkbox v-model="muted" id="audio">Muted</b-form-checkbox>
      </b-col>
    </b-row>

    <b-row v-if="editObject.path !== undefined">
      <h5>Path</h5>
      <b-form-input id="name" :value="editObject.path" @change="dispatchPath"></b-form-input>
    </b-row>
    <b-row v-else>
      <h5>Name</h5>
      <b-form-input id="name" :value="editObject.name" @change="dispatchPath"></b-form-input>
    </b-row>

    <b-row v-if="editObject.type === 'folder'">
      <h5>Playback order</h5>
      <b-form-select v-model="sorting" :options="sortingOptions" size="sm" class="mt-3"></b-form-select>
    </b-row>
    <b-row v-if="editObject.type === 'input'">
      <h5>Input</h5>
      <b-form-input id="input" :value="editObject.input" @change="dispatchInput" type="number"></b-form-input>
    </b-row>
    <b-row v-if="editObject.type === 'input'">
      <h5>Duration</h5>
      <b-form-input id="duration" :value="editObject.duration" @change="dispatchDuration" type="number"></b-form-input>
    </b-row>

    <b-row>
      <b-col>
        <h5>Days</h5>
        <b-form-checkbox
          v-for="(day, index) in ['su', 'mo', 'tu', 'we', 'th', 'fr', 'sa']"
          :key="index"
          :checked="days[index]"
          @change="toggleDay(index)"
          :id="day"
          >{{ day.toUpperCase() }}</b-form-checkbox
        >
      </b-col>
    </b-row>

    <b-row>
      <b-col>
        <h5>Weeks</h5>
        <b-form-input id="week" :value="weeks" @change="dispatchWeeks" placeholder="e.g. 1,2,3..."></b-form-input>
        <b-form-text>Week numbers (separate with ,)</b-form-text>
      </b-col>
    </b-row>

    <b-row>
      <b-col>
        <h5>Dates</h5>
        <b-input-group v-for="(date, index) in dates" :key="index">
          <b-input-group-prepend>
            <b-btn v-on:click.prevent="removeDate(index)">
              <font-awesome-icon icon="trash"></font-awesome-icon>
            </b-btn>
          </b-input-group-prepend>
          <!-- <b-form-input type="date" v-bind:value="date[0]" v-on:change="updateDate($event, index, 0)"></b-form-input>
          <b-form-input type="date" v-bind:value="date[1]" v-on:change="updateDate($event, index, 1)"></b-form-input>-->
          <b-form-datepicker
            type="date"
            v-bind:value="date[0]"
            :state="date | validDates"
            v-on:input="updateDate($event, index, 0)"
          ></b-form-datepicker>
          <b-form-datepicker
            type="date"
            v-bind:value="date[1]"
            :state="date | validDates"
            v-on:input="updateDate($event, index, 1)"
          ></b-form-datepicker>
        </b-input-group>

        <b-input-group>
          <!-- <b-form-input type="date" v-model="newDate[0]"></b-form-input> -->
          <b-form-datepicker v-model="newDate[0]" :state="newDate | validDatesExplicit"></b-form-datepicker>
          <!-- <b-form-input type="date" v-model="newDate[1]"></b-form-input> -->
          <b-form-datepicker v-model="newDate[1]" :state="newDate | validDatesExplicit"></b-form-datepicker>
          <b-input-group-append>
            <b-btn variant="primary" v-on:click.prevent="addDate()">Add daterange</b-btn>
          </b-input-group-append>
        </b-input-group>
      </b-col>
    </b-row>

    <b-row>
      <b-col>
        <h5>Times</h5>
        <b-input-group v-for="(time, index) in times" v-bind:key="index">
          <b-input-group-prepend>
            <b-btn @click.prevent="removeTime(index)">
              <font-awesome-icon icon="trash"></font-awesome-icon>
            </b-btn>
          </b-input-group-prepend>
          <b-form-input :value="time" @change="updateTime($event, index)"></b-form-input>
        </b-input-group>

        <b-input-group>
          <b-form-input v-model="newTime"></b-form-input>
          <b-input-group-append>
            <b-btn variant="primary" @click.prevent="addTime">Add time</b-btn>
          </b-input-group-append>
        </b-input-group>
      </b-col>
    </b-row>
  </b-container>

  <!-- <br /><hr />
            <div class="row">
                <div class="col s6">
                    <a class="waves-effect waves-teal btn-flat" style="width: 100%" v-on:click.prevent="setProperties">Cancel</a>
                </div>
                <div class="col s6">
                    <a class="waves-effect waves-light btn" style="width: 100%" v-on:click.prevent="saveObject">Save</a>
                </div>
  </div>-->
</template>

<script>
export default {
  // @todo: init date picker
  name: 'edit-schedule',
  computed: {
    editObject: function () {
      console.log('this is new editobj')
      let obj = this.$store.getters.scheduleEntryById(this.id)
      if (!obj.days) {
        obj.days = []
      }
      return obj
    },
    times: function () {
      return this.$store.getters.scheduleEntryById(this.id).times
    },
    dates: function () {
      return this.$store.getters.scheduleEntryById(this.id).dates
    },
    id: function () {
      return this.$route.params.id
    },
    weeks: function () {
      if (this.editObject && this.editObject.weeks) {
        return this.editObject.weeks.join(', ')
      } else {
        return ''
      }
    },
    days: function () {
      const days = []
      for (let i = 0; i < 7; i++) {
        days[i] = new Set(this.editObject.days || []).has(i)
      }
      return days
    },
    sorting: {
      get: function () {
        return this.$store.getters.scheduleEntryById(this.id).sort
      },
      set: function (value) {
        this.$store.dispatch('setSorting', { _id: this.id, value })
      },
    },
  },
  data: function () {
    return {
      muted: false,
      newTime: '',
      newDate: ['', ''],
      sortingOptions: [
        { text: 'Name Ascending', value: 'name_asc' },
        { text: 'Name Descending', value: 'name_desc' },
        { text: 'Date Ascending', value: 'date_asc' },
        { text: 'Date Descending', value: 'date_desc' },
      ],
    }
  },
  methods: {
    getDay: function (day) {
      console.log(day, this.$store.getters.scheduleEntryById(this.id).days[day])
      return this.$store.getters.scheduleEntryById(this.id).days[day] || false
    },
    toggleDay: function (value) {
      console.log(value)
      this.$store.dispatch('toggleDay', { _id: this.id, day: value })
    },

    dispatchWeeks: function (value) {
      if (/([0-9]+)(,[0-9]+)*/.test(value)) {
        value = value.split(',').map((o) => Number(o))
        this.$store.dispatch('setWeeks', { _id: this.id, value })
      } else {
        if (value === '') {
          this.$store.dispatch('setWeeks', { _id: this.id, value: [] })
        }
        // @todo: invalidate.
      }
    },

    dispatchPath: function (value) {
      this.$store.dispatch('setPath', { _id: this.id, value })
    },

    dispatchInput: function (value) {
      this.$store.dispatch('setInput', { _id: this.id, value: Number(value) })
    },

    dispatchDuration: function (value) {
      this.$store.dispatch('setDuration', { _id: this.id, value: Number(value) })
    },

    addDate: function () {
      let dates = this.newDate

      if (dates[0] !== '' && dates[1] !== '' && dates[0] < dates[1]) {
        this.$store.dispatch('addDateEntry', { _id: this.id, dateEntry: dates })
        dates = ['', '']
      }
    },
    updateDate: function (val, index, type) {
      // where type = 0 for start and 1 for end
      this.$store.dispatch('updateDateEntry', { _id: this.id, dateEntry: index, type: type, date: val })
    },
    removeDate: function (index) {
      this.$store.dispatch('deleteDateEntry', { _id: this.id, dateEntry: index })

      // if (this.editObject.dates[index])
      //     this.editObject.dates.splice(index, 1)
    },
    isDateValid(dates, explicit = false) {
      const valid = dates[0] <= dates[1]

      if (!valid) {
        return false
      } else if (explicit) {
        return true
      } else {
        return undefined
      }
    },

    addTime: function (event) {
      if (/\b(\d){2}(:){1}(\d){2}(:){1}(\d){2}\b/.test(this.newTime)) {
        this.$store.dispatch('addTime', { _id: this.id, time: this.newTime })
        this.newTime = ''
      }
    },
    updateTime: function (val, index) {
      if (/\b(\d){2}(:){1}(\d){2}(:){1}(\d){2}\b/.test(val)) {
        this.$store.dispatch('updateTime', { _id: this.id, index: index, time: val })
      } else {
        val = this.times[index]
      }
    },
    removeTime: function (index) {
      if (this.editObject.times[index]) {
        this.$store.dispatch('deleteTime', { _id: this.id, index: index })
      }
    },

    saveObject: function () {
      let props = this.editObject
      var list = this.schedule[this.path[0]]
      for (let i = 1; i < this.path.length; i++) {
        if (list.type === 'group') {
          list = list.children[this.path[i]]
        }
      }
      console.log(list)

      list.days = props.days.length > 0 ? props.days : undefined

      if (props.weeks !== '') {
        list.weeks = []
        let weeks = props.weeks.split(',')
        for (let week of weeks) {
          list.weeks.push(week.trim())
          console.log(list.weeks)
        }
      } else if (list.weeks) {
        list.weeks = undefined
      }

      if (props.dates.length > 0) {
        list.dates = []
      } else {
        list.dates = undefined
      }
      for (let dates of props.dates) {
        dates[0] = dates[0].split('-').reverse().join('-')
        dates[1] = dates[1].split('-').reverse().join('-')
        list.dates.push(dates)
      }

      list.times = props.times.length > 0 ? props.times : undefined

      if (props.type !== 'group') {
        list.path = props.path ? props.path : ''
      } else {
        list.name = props.name !== undefined ? props.name : ''
      }
    },

    deleteObject: function () {
      let _id = this.id

      let returnAndDeleteById = (parent) => {
        if (parent.children) {
          for (let child of parent.children) {
            if (child._id === _id) {
              // child we are editing
              // delete:
              this.$store.dispatch('deleteEntry', { _id: this.id })

              // navigate:
              this.$router.push({ path: '/schedule/' + (parent._id === 'MAIN_ENTRY' ? '' : parent._id + '/edit') })

              return parent
            } else if (child.type === 'group') {
              // not the  child we are editing, but child is a group, therefore might contain what we are editing
              let res = returnAndDeleteById(child) || null

              if (res) {
                return res
              }
            }
          }
        }
      }

      returnAndDeleteById({ children: this.$store.state.schedule, _id: 'MAIN_ENTRY' })

      // this.$store.dispatch('deleteEntry', {_id: this.id});

      //
    },
  },
  filters: {
    validDates(dates) {
      if (!dates[0] || !dates[1]) return false
      const valid = dates[0] <= dates[1]

      if (!valid) {
        return false
      } else {
        return undefined
      }
    },
    validDatesExplicit(dates) {
      if (!dates[0] && !dates[1]) return undefined

      return dates[0] <= dates[1]
    },
  },
  watch: {
    muted: function (val) {
      const oldVal = this.editObject.audio === false
      if (oldVal !== val) this.$store.dispatch('setMuted', { _id: this.id, muted: val })
    },
    editObject: function (v) {
      this.muted = v.audio === false
    },
  },
}
</script>
