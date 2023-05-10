<template>
  <div id="app">
    <div>
      <div>
        <p>
          <label>URL:</label>
          <input type="text" v-model="url" placeholder="http://" />
        </p>

        <p>
          <label>Format:</label>
          <select v-model="format">
            <option value="plain">Plain</option>
            <option value="json">JSON</option>
          </select>
        </p>

        <p>
          <label>
            <input type="checkbox" v-model="includeCredentials" />
            Include credentials?
          </label>
        </p>

        <p>
          <button :disabled="url.length === 0" @click.prevent="connect">
            Connect
          </button>
          <button @click.prevent="disconnect">Disconnect</button>
          <button @click.prevent="clear">Clear</button>
        </p>
      </div>

      <handlers :handlers.sync="handlers" />
    </div>

    <div>
      <LogDisplay :logs="logs" />
    </div>
  </div>
</template>

<script lang="ts">

// @ts-nocheck
import Vue from 'vue'

import { SSEClient } from 'vue-sse/types'
import Handlers from './components/Handlers.vue'
import LogDisplay from './components/LogDisplay.vue'
import axios from 'axios'
import { now } from './utils'

let client: SSEClient | null

export default Vue.extend({
  name: 'App',
  components: {
    Handlers,
    LogDisplay
  },
  data () {
    return {
      dataSource: [],
      url: 'http://localhost:8000',
      includeCredentials: true,
      format: 'plain',
      handlers: [
        {
          event: 'message',
          color: '#60778e'
        },
        {
          event: 'time',
          color: '#778e60'
        }
      ],
      logs: [] as [string, string, string][]
    }
  },
  mounted () {
    this.connect()
  },
  methods: {
    connect () {
      // try disconnecting, just in case
      this.disconnect()

      this.log(`[info] connecting to ${this.url}`, 'system')

      // create the client with the user's config
      client = this.$sse.create({
        forcePolyfill: true,
        polyfill: true,
        polyfillOptions: {
          headers: {

            HeaderTest: 'blabla'

          }

        },
        url: this.url,
        includeCredentials: this.includeCredentials,
        format: this.format
      })

      // add the user's handlers
      this.handlers.forEach(h => {
        client!.on(h.event, data => {
          console.log(data, '111')
          if (data === 'table') {
            this.getData()
          }
          // eslint-disable-line
          this.log(data, h.color)
        })
      })

      client!.on('error', () => {
        // eslint-disable-line
        this.log(
          '[error] disconnected, automatically re-attempting connection',
          'system'
        )
      })

      // and finally -- try to connect!
      client!
        .connect() // eslint-disable-line
        .then(() => {
          this.log('[info] connected', 'system')
        })
        .catch(() => {
          this.log('[error] failed to connect', 'system')
        })
    },

    getData () {
      axios
        .post(
          'https://pilot.ezgolf.vn/api/AR_RSMS01/Process',
          {
            Lang: '1000000',

            Token:
              'Jj5a2mC3JSWtgjjNfxX6ldOEilb2aFeyJBY3RpdmVTaXRlSWQiOiI3RkRCQzU3My1ERjUzLTQ5QjYtOTE1Mi00OTRDRjdBQUM2NkYiLCJIb3RlbElkIjoiZXoyIiwiVXNlcklkIjoiOGVlNGNhYzYtYTdlZC00NGM1LWEyMDUtYjg5OTMzOTk5ZjVmIiwiVXNlck5hbWUiOiJlekFyIiwiQXV0aGVuVG9rZW4iOiJNMDgxTTNwUWRHRjVaamRUTjBwTU1TMW1abU5JTFhoeVluUjNSVGRSVERsSFkwRkllR3BPUnpkbk1qRXlhV1IxYkZCc1RtcEpOVE5ETm5kQ1MySmxiMWd6TUhobGVIcDBWemh6TkdsRWJtMW1abHByVDA5T2JVcGFXSEJEY1hkUWRFSXdPUzAyUTB0M01ESkNZWFJoTFZObVFWbHhlVGxXUWtwblFrWlRPVWhKZUY5bFQyZHlVMVJMTjA1M09YTlFjVTlGYVhwaFlrTjBaMWRHVkVSZllXMVlRbEYxZFV0WWQySXpZV05zYTA5bk0xVnBUbDk1TVV4aVJVMHlVa041T0hKclpWRm5OV3BqVUdGcVIweE9WMGxNYW5CdVYzaHJPWGRMTldSblJ5MHhTekZXV0ZsaldGOHRhV1JqVkMxdVIwVjVOVVp3YUhaUFZrUkZkVVI1TjBJd1VUWjRVbkV4ZG5WZmVVbHhPRTFoWDBkV1JrczNNMmxQV0c5VVRVUTFWakpJZGswMVlucE9hbkp5TnkxcFkwMVJYMGxPUjFkNmNsTkRPVzUzZVRCMlNFRlRVemN4UVhCclJsazFTWFE1VXpsbWJUWlJUVzEyTVROSGJIRmZXWEJoU2xkRFlVMXhSbXB0VUhRdFJVVkljM0JFV1RkclEzaE5jRlJ0Wm5OaVpuSlplbEpJWDJWeU4wRmlRazU2ZG5kWGQyODRlRnBOTWxGcFgxUm5iakpHUkVSellsSnlSalZOZEd3eGJUZEZVbU5ZTTBrMU9TMHRaWEF6VDNwTlpGYzBXVTlOUVhJMFdubEdTVFZOZVZaSmIzbzRZMmRXUkZseFlYZGxjVlZrVlMxNFJEbDZVMWh5VlU1UWJWbFZSMnhJZHpCdGVFeHRNek5OVjBGemJVeGFSVVZvT0cxalIxSkZRbGt3YWpoRVNESXlXbWxqTW5CaU1tRldjVXRsTkZSVWIxb3hkVVpHU201RFRVUlNPVEEzY2toNVJGQTVTRWxuYzNaQ1lsUTJiVmxhZVcxUlFUQTRPVU0yWHkxMGVYTTVSVzQ1Y1VSbWNIaDNORFpNY2pVMVNWOWZXVkp2YlhWVWQwWnJNMVpxZVdadlNWYzFVakJtWkZnNVZIa3hiMDAyTFVOWlVURmtPVmROYUdKSldYRlphMkpIZG05TGJVMUxOVGh1YVVkdGNXdE5Oa0U1V25CWVVEaHZPRTl5ZGpaR1VHTkxhbmRHTVZoRGQwUTNSM1pEYm5SbGVqVkxVWHBvTVVWT1lsWmhkR2hrVlVJMU9WVk5ia2RoWW5GVGMySnpORkpsVnpKaVVXTldTbEo1UVU1UGNITkRiRlJNVGpkUk0zRkJWalpXV2xwVWRHNXVZVkpRTkRaSGFrOWFWVkk1VEZaaFZFcEpPV1Y0ZW1wd1NVZHNVRFZ1WVRab04ydEdjR3B6U0Y5NU9VbzFhbU5VV1hKc1RGVjRjRlZsV1ZSNWJXazBXVXBPY2t0a00yZGtTMVpuZVUxcWVXeFRVbUZaTFRVMmRHOWFURzFGT1hGd1ZVUmFWMk14UkU1WlVtWldUelpuUlVwaGNubEpjM0E1TkhaTWFtOWhTemsxYTBodGRWbEVSa3hzZGxOcFJVcHpVSFpOZFhGR2ExRkZNQzFpU25GWVJHODVVbEEwIiwiVmVyc2lvbiI6IjEuMC4wIiwiSXNzdWVkIjoiMjAyMy0wNS0xMFQwODo0MjowMi42NjkyODYxKzA3OjAwIiwiRXhwaXJlZCI6IjIwMjMtMDUtMTBUMTA6NDI6MDIuNjY5Mjg5MSswNzowMCIsIlBVSyI6ImI1NDRmNzgzLWUwZmQtNGJiNi1hMTlmLTU0MzNmZTQxNDlkMSIsIkNoZWNrU3VtIjoiMTItREYtMzAtOTEtQ0EtNEItMjMtQTQtMjMtQ0EtQzYtMkEtNjAtQTktQTYtM0EiLCJTaXRlIjpbeyJTaXRlSWQiOiI3RkRCQzU3My1ERjUzLTQ5QjYtOTE1Mi00OTRDRjdBQUM2NkYiLCJTaXRlQ29kZSI6IiIsIlNpdGVOYW1lIjoiVGhlIEZpdmUgVmlsbGFzIFx1MDAyNiBSZXNvcnQgTmluaCBCXHUwMEVDbmgiLCJJbkFjdGl2ZSI6ZmFsc2UsIlBlcm1pc3Npb24iOlsiQVJfREFBMDEiLCJSQ0wwMSIsIlJDT00wMiIsIlJMQTAxIiwiUlRDMDEiLCJBUl9EQ0wwMSIsIkFSX0REQTAxIiwiQVJfRFBBMDEiLCJBUl9EUEFEMDEiLCJBUl9EU01TMDEiLCJBUl9IU0QwMSIsIkFSX0hTUDAxIiwiQVJfSUFBMDEiLCJSQ0xBUiIsIlJTT0FSIiwiQVJfSUNMMDEiLCJBUl9JREEwMSIsIkFSX0lQQTAxIiwiQVJfSVBBRDAxIiwiQVJfSVBTTVMwMSIsIkFSX0lTTVMwMSIsIkFSX1JBQTAxIiwiQVJfUkFBMDIiLCJBUl9SQVQwMSIsIkFSX1JDTDAxIiwiUlNPMDEiLCJBUl9SQ0wwMiIsIkFSX1JQQTAxIiwiQVJfUlBBMDIiLCJBUl9SUEFEMDEiLCJBUl9SUEQwMSIsIkFSX1JQRDAyIiwiQVJfUlNNUzAxIiwiQVJfUlNNUzAyIiwiQVJfVUFBMDEiLCJBUl9VQ0wwMSIsIkFSX1VEQTAxIiwiQVJfVVBBMDEiLCJBUl9VUEFEMDEiLCJBUl9VU01TMDEiLCJNRFIwMSIsIlJCS0QwMSIsIlJEUjAxIiwiUkhTMDIiLCJSUFRBUiIsIlJTVjAyIiwiUkJLRDA0IiwiUkJLRDJBIiwiUkJLRDJCIiwiUkJLRDJEIiwiUlJDMDEiLCJSU1ZBUiIsIklCS0QwMSIsIklCS0QwMiIsIlJCS0QyQyIsIlJCS0QzIiwiVUJLRDAxIiwiVUJLRDAyIiwiVUJLRDMiLCJVQktEOSIsIkRCS0QwMSIsIkRCS0QwMiIsIklDSUQwMSIsIklDSUQwMiIsIklDSUQwMyIsIk1CSzAxIiwiTUJLMDIiLCJSQ0wwMyIsIlJDTzAxIiwiUkNPTTAxIiwiUkYwMSIsIlJHQzAxIiwiUlRFMDEiLCJSVFMwMSIsIlJUVDAxIiwiUkJLMDJBIiwiUkJLMDJCIiwiUkhTMDEiLCJJQkswMUEiLCJJRU0wMSIsIklFTTAyIiwiVUJLMDYiLCJJQ0kwMyIsIlVTVTA1IiwiUkJLMDQiLCJSQkswNSIsIlJCSzA2IiwiUkJLMDciLCJSQkswOCIsIlJSQTA0IiwiUlJBMDRBIiwiSUJLMDEiLCJSQksxMiIsIlJUQjAxIiwiUlRCMDIiLCJSVENBUiIsIlJURTAzIiwiVUJLMDkiLCJVQksxMiIsIlVCSzEzIiwiSUJLMDIiLCJJTUUwMSIsIlJCSzAyQyIsIlJCSzAzIiwiUkJLMDkiLCJSQksxMCIsIlJDQTA0IiwiUk1FMDEiLCJSTUUwNSIsIlVCSzAxIiwiVUJLMDIiLCJVQkswMkIiLCJVQkswMyIsIlVCSzAzQyIsIlVCSzA0IiwiVUJLMDUiLCJVQkswNyIsIlVCSzEwIiwiVUJLMTEiLCJVQksxNSIsIlVCSzE2IiwiVU1FMDEiLCJEQkswMSIsIkRCSzAyIiwiSUNJMDEiLCJVQ0kwMSIsIklDSTAyIiwiUkJLMDJEIiwiVUJLMDgiLCJNQktXTDAxIiwiUkJLV0wxIiwiVUJLV0wxIiwiREJLV0wxIiwiTUJSMDEiLCJSQktSVDEiLCJSSFMwMyIsIlJTVTAzIiwiUkJLUlQyIiwiVUJLUlQxIiwiSUJLUlQxIiwiSUNJUjAyIiwiVU1FMDQiLCJVTUUwNSIsIklDSVIwMyIsIkRCS1JUMDEiLCJNQ0EwMyIsIlJDQUcwMSIsIlJOQTAxIiwiUkNTMDEiLCJVQ0EwMSIsIlVDQTA0IiwiRENBMDEiLCJVQ0EwMiIsIlVDQTAzIiwiTUNBMDIiLCJSQ0FDMDEiLCJSQ1dTMDEiLCJSQ0FDMDIiLCJJQ0FDMDEiLCJSQ0EwMSIsIlVDQUMwMSIsIlVDQUMwNCIsIkRDQUMwMSIsIk1DQTAxIiwiUkNBRTAxIiwiSUNBRzAxIiwiUkNBVjAxIiwiVUNBVjAxIiwiSUNBRTAxQSIsIlVDQUUwMSIsIkRDQUcwMSIsIlVDQUcwMSIsIlVDQUcwMiIsIlVDQUcwMyIsIlJDQTAzIiwiUkNBMDIiLCJJQ0EwMSIsIk1DTzAxIiwiUkhUMDEiLCJSQ08wMiIsIkRDRjAxIiwiRENGMDIiLCJEQ0ZWMDEiLCJEUkNGMDIiLCJEUkNGVjAxIiwiSUNGMDEiLCJJQ0ZWMDEiLCJJQ08wMSIsIklSQ0YwMSIsIklSQ0ZWMDEiLCJNUk8wMSIsIlJDRjAxIiwiUkNGMDIiLCJSQ0ZWMDEiLCJSQ0ZWMDIiLCJSUkNGMDEiLCJSUkNGMDIiLCJSUkNGVjAxIiwiUlJDRlYwMiIsIlJSTzAxIiwiVUNGMDEiLCJVQ0ZWMDEiLCJVUkNGMDEiLCJVUkNGVjAxIiwiVVJPMDEiLCJSSE8wMSIsIlVDTzAxIiwiRENPMDEiLCJVQ08wMiIsIlVDTzAzIiwiUkRSMDIiLCJJRFIwMSIsIlVEUjAxIiwiRERSMDEiLCJVRFIwMiIsIlVEUjAzIiwiTUdDMDEiLCJSR0MwMiIsIklHQzAxIiwiVUdDMDEiLCJER0MwMSIsIlVHQzAyIiwiVUdDMDMiLCJNTUMwMSIsIlJNRUMwMSIsIlJNRUMwMiIsIklNRUMwMSIsIlJSQTAxIiwiUlJBMDUiLCJVTUVDMDEiLCJETUVDMDEiLCJVTUVDMDIiLCJVTUVDMDMiLCJNTUUwMSIsIlJNRTAzIiwiRFNVMDEiLCJSU1UwMiIsIlVTVTAyIiwiVVNVMDMiLCJVU1UwNCIsIlJNRTAyIiwiRE1FMDEiLCJVTUUwMiIsIlVNRTAzIiwiUk1FMDQiLCJSTUUwNEEiLCJSTUVIMDEiLCJSU1UwMSIsIklQTTA2IiwiVU1FMDRBIiwiSVNVMDEiLCJVU1UwMSIsIkRSQ0UwMSIsIkRSQ1owMSIsIklSQ0UwMSIsIklSQ1owMSIsIk1SQzAxIiwiTVJMUEQwMSIsIlJMUEQwMSIsIlJSQ0MwMCIsIlJSQ0MwMFAiLCJSUkNFMDEiLCJSUkNFMDIiLCJSUkNTMDMiLCJSUkNUMDQiLCJSUkNaMDEiLCJSUkNaMDIiLCJSUkYwMSIsIlVSQ0MwMSIsIlVSQ0UwMSIsIlVSQ1AwMSIsIlVSQ1NOMSIsIlVSQ1QwMSIsIlVSQ1RDMSIsIlVSQ1owMSIsIlVSRjAxIiwiVVJHQzAxIiwiTVBBMDEiLCJSUEEwMSIsIlJQQTAyIiwiUlNEMDEiLCJSU0QwMiIsIkRTRDAxIiwiSVBBMDEiLCJJUEEwMiIsIklQQTAzIiwiSVNEMDEiLCJVU0QwMSIsIlVQQTAzIiwiVVBBMDQiLCJVUEEwNSIsIlVQQTA2IiwiRFBBMDEiLCJEUEEwMiIsIkRQQTAzIiwiVVBBMDEiLCJVUEEwMiIsIlJQRjAxRiIsIlJQTTAxIiwiSVBNMDJEIiwiVVBNMDJEIiwiUkNVMDEiLCJSUEYwMSIsIlJQRjAyIiwiUlBNMDJEIiwiRFBNMDJEIiwiVVBNMDciLCJSQkFBUiIsIlJJVjAxIiwiUlJQREYiLCJSQktGMSIsIklQTTAxIiwiSVBNMDFGIiwiSVBSMDEiLCJSTUUwMUYiLCJEUE0wNyIsIlVQTTAxIiwiVVBNMDIiLCJEUE0wMiIsIkRQTTA5IiwiRFBSMDEiLCJSUE0wMiIsIklQTTAyIiwiUlBNMDJNIiwiSVBNMDJSIiwiUlBNMDMiLCJEUE0wNCIsIlJJVjAyIiwiVVBNMDgiLCJJUE0wMyIsIlVQTTA0IiwiRFBNMDMiLCJEUE0xMCIsIlJQTTA0IiwiSVBNMDQiLCJEUE0wMSIsIkRQTTA4IiwiVVBNMDMiLCJNUEEwMiIsIklQTTA1IiwiVVBNMDYiLCJSUE0wNiIsIkRQTTA2IiwiUklWMDMiLCJNUkEwMSIsIlJSQTAyIiwiUlJBMDJBIiwiSVJBMDEiLCJJUkEwMiIsIklSQTAzIiwiUlJBMDMiLCJVUkEwMyIsIlVSQTA0IiwiVVJBMDUiLCJEUkEwMSIsIkRSQTAyIiwiRFJBMDMiLCJVUkEwMSIsIlVSQTAyIiwiTVJCMDdBIiwiTVJQMDEiLCJSQjA3QSIsIlJDVUFSIiwiUlJDQzAxICIsIlJSQ0MwMiIsIk1SQjA3QiIsIlJCMDdCIiwiTVJDUzA2IiwiUkNTMDYiLCJNUkYwNyIsIk1SRjA3QSIsIk1SRjA3QyIsIlJGMDciLCJSRjExIiwiTVJGMDgiLCJSRjA4IiwiTVJGMDkiLCJSRjA5IiwiTUZCLTAxIiwiTUZCLTAyIiwiUkZCMDIiLCJNR08tMDIiLCJNR08tMDMiLCJNR08tMDQiLCJNSFYtMDEiLCJNSFYtMDIiLCJNSFYtMDQiLCJSUkhWMDQiLCJNS1QtMDEiLCJSUkNDWjAzIiwiUlJDWjAzIiwiTUtULTA0IiwiUktUMDQiLCJNS1QtMDkiLCJSRjEyIiwiTUtULTEwIiwiUkYxMkQiLCJNTFQtMDEiLCJNTFQtMDIiLCJNTFQtMDMiLCJNTFQtMDQiLCJNTFQtMDUiLCJNTFQtMDciLCJNVE4tMDEiLCJJQktTQTEiLCJNU1QwMSIsIlJCS1NBMSIsIlJCS1NBMyIsIlJCS1NBMiIsIlVDQUMwMiIsIklUVDAxIiwiTVRFMDEiLCJEVFQwMSIsIlVUVDAxIiwiVVRUMDIiLCJSVEUwMiIsIklURTAxIiwiVVRFMDEiLCJEVEUwMSIsIlVURTAyIiwiVVRFMDMiXSwiVGltZVpvbmUiOiJBc2lhL0Jhbmdrb2siLCJUaW1lWm9uZU9mZnNldCI6NDIwLCJIb3RlbElkIjoiZXoyIn1dLCJDYWRkeUlkIjpudWxsfQ==3QJj5a2mC3JSWtgjjN'
          },
          {
            headers: {
              'Custom-header': 'Blabla'
            }
          }
        )
        .then(res => {
          console.log(res, '1500')
          if (res.data.Status === '200') {
            this.dataSource = res.data.Data.ListPaymentSMS
          }
        })
    },
    disconnect () {
      if (client) {
        client.disconnect()
        client = null
        this.log('[info] disconnected', 'system')
      }
    },

    clear () {
      this.logs = []
    },

    log (message: string, color: string) {
      this.logs.push([now(), message, color])
    }
  },
  beforeDestroy () {
    this.disconnect()
  }
})
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
}

h2 {
  display: inline;
}

button,
input,
select {
  font: inherit;
}

#app {
  background: #dddddd;
  font-family: sans-serif;
  min-height: 100vh;
  position: relative;
}

@media (min-width: 768px) {
  #app {
    display: flex;
  }

  #app > div {
    padding: 1rem;
  }

  #app > div:first-child {
    text-align: right;
  }

  #app > div:last-child {
    flex: 1;
  }
}
</style>
