<template>
  <v-row>
    <v-col>
      
     
      <v-dialog v-model="dialog" max-width="500">
        <v-card>
          <v-container>
            <v-form @submit.prevent="addEvent">
              <v-text-field v-model="name" type="text" label="title"></v-text-field>
              <v-text-field v-model="details" type="text" label="patient name"></v-text-field>
              <v-text-field v-model="start" type="datetime-local" label="start"></v-text-field>
              <v-text-field v-model="end" type="datetime-local" label="end"></v-text-field>
              <v-text-field v-model="color" label="color"></v-text-field>
              <v-btn type="submit" color="success" class="mr-4" @click.stop="dialog = false">
                submit
              </v-btn>
            </v-form>
          </v-container>
        </v-card>
      </v-dialog>

      <v-sheet height="64">
        <v-toolbar
          flat
        >
          <v-btn
            fab
            text
            small
            color="grey darken-2"
            @click="prev"
          >
            <v-icon small>
              mdi-chevron-left
            </v-icon>
          </v-btn>
          <v-btn
            fab
            text
            small
            color="grey darken-2"
            @click="next"
          >
            <v-icon small>
              mdi-chevron-right
            </v-icon>
          </v-btn>
          <!-- <v-toolbar-title v-if="$refs.calendar">
            {{ $refs.calendar.title }}
          </v-toolbar-title> -->
           <v-btn
            color="primary"
            style="margin:20px"
            dark
            @click.stop="dialog = true"
          >
            Add new appointment
          </v-btn>

        </v-toolbar>
      </v-sheet>

      <v-sheet height="600">
        <v-calendar
          ref="calendar"
          :now="today"
          :value="today"
          :events="events"
          color="primary"
          type="week"
          :weekdays="[ 1, 2, 3, 4, 5]"
          :event-overlap-threshold="30"
          :event-color="getEventColor"
          v-model="focus"
          @click:event="showEvent"
          @click:more="viewDay"
          @click:date="viewDay"
          @change="updateRange"

        ></v-calendar>
         <v-menu
          v-model="selectedOpen"
          :close-on-content-click="false"
          :activator="selectedElement"
          offset-x
        >
          <v-card
            color="grey lighten-4"
            min-width="350px"
            flat
          >
            <v-toolbar
              :color="selectedEvent.color"
              dark
            >
              <v-btn icon>
                <v-icon>mdi-pencil</v-icon>
              </v-btn>
              <v-toolbar-title v-html="selectedEvent.name"></v-toolbar-title>
              <v-spacer></v-spacer>
              <v-btn icon>
                <v-icon>mdi-heart</v-icon>
              </v-btn>
              <v-btn icon>
                <v-icon>mdi-dots-vertical</v-icon>
              </v-btn>
            </v-toolbar>
            <v-card-text>
              <span v-html="selectedEvent.details"></span>
            </v-card-text>
            <v-card-actions>
              <v-btn
                text
                color="secondary"
                @click="selectedOpen = false"
              >
                Cancel
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-menu>
      </v-sheet>
    </v-col>
  </v-row>
</template>

<script>

  export default {
    data: () => ({
      focus: '',
      name: null,
      details: null,
      start: null,
      end: null,
      color: null,
      selectedEvent: {},
      selectedElement: null,
      selectedOpen: false,
      today: new Date().toISOString().slice(0, 10),
      events: [
        {
          name: 'Afpsraak met Sinan',
          start: '2021-02-02 10:00',
          end: '2021-02-02 11:30',
          color: 'blue',
          details: 'Patient001',
        },{
          name: 'Afpsraak met Topicus',
          start: '2021-02-03 14:00',
          end: '2021-02-03 15:00',
          color: 'green',
          details: 'Patient002',
        }
      ],
      dialog: false,
    }),
    mounted () {
      this.$refs.calendar.scrollToTime('08:00')
    },
    computed: {
      title () {
        const { start, end } = this
        if (!start || !end) {
          return ''
        }
        const startMonth = this.monthFormatter(start)
        const endMonth = this.monthFormatter(end)
        const suffixMonth = startMonth === endMonth ? '' : endMonth
        const startYear = start.year
        const endYear = end.year
        const suffixYear = startYear === endYear ? '' : endYear
        const startDay = start.day + this.nth(start.day)
        const endDay = end.day + this.nth(end.day)
        switch (this.type) {
          case 'month':
            return `${startMonth} ${startYear}`
          case 'week':
          case '4day':
            return `${startMonth} ${startDay} ${startYear} - ${suffixMonth} ${endDay} ${suffixYear}`
          case 'day':
            return `${startMonth} ${startDay} ${startYear}`
        }
        return ''
      },
      monthFormatter () {
        return this.$refs.calendar.getFormatter({
          timeZone: 'UTC', month: 'long',
        })
      },
    },
    methods: {
      viewDay ({ date }) {
        this.focus = date
        this.type = 'day'
      },
      getEventColor (event) {
        return event.color
      },
      setToday () {
        this.focus = this.today
      },
      prev () {
        this.$refs.calendar.prev()
      },
      next () {
        this.$refs.calendar.next()
      },
      async addEvent () {
        this.start = await new Date(this.start).toISOString().substring(0,16)
        this.end =  await new Date(this.end).toISOString().substring(0,16)
                  this.events.push({name: this.name,
                              details: this.details,
                              start: this.start,
                              end: this.end,
                             color: this.color})

      },
      showEvent ({ nativeEvent, event }) {
        const open = () => {
          this.selectedEvent = event
          this.selectedElement = nativeEvent.target
          setTimeout(() => this.selectedOpen = true, 10)
        }
        if (this.selectedOpen) {
          this.selectedOpen = false
          setTimeout(open, 10)
        } else {
          open()
        }
        nativeEvent.stopPropagation()
      },
      updateRange ({ start, end }) {
        // You could load events from an outside source (like database) now that we have the start and end dates on the calendar
        this.start = start
        this.end = end
      },
      nth (d) {
        return d > 3 && d < 21
          ? 'th'
          : ['th', 'st', 'nd', 'rd', 'th', 'th', 'th', 'th', 'th', 'th'][d % 10]
      },
    },
  }
</script>

<style scoped>
.my-event {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  border-radius: 2px;
  background-color: #1867c0;
  color: #ffffff;
  border: 1px solid #1867c0;
  font-size: 12px;
  padding: 3px;
  cursor: pointer;
  margin-bottom: 1px;
  left: 4px;
  margin-right: 8px;
  position: relative;
}

.my-event.with-time {
  position: absolute;
  right: 4px;
  margin-right: 0px;
}
</style>