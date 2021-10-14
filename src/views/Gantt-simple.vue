<template lang="pug">
v-layout
  v-layout(v-if='loaded')
    #svg-root
    canvas#canvas-root(style='visibility: hidden')
    pre#str
  v-layout(v-else)
    v-progress-circular(indeterminate :width="7" :size="70")
</template>
<script lang="ts" defer>
import Vue from 'vue'
import Component from 'vue-class-component'
import { SVGGantt, CanvasGantt, StrGantt } from 'gantt'

import { getTasks } from '@/utils/api'

@Component({})
export default class GanttSimple extends Vue {
  data = [
    {
      id: 1,
      type: 'group',
      text: '1 Waterfall model',
      start: new Date('2018-10-10T09:24:24.319Z'),
      end: new Date('2018-12-12T09:32:51.245Z'),
      percent: 0.71,
      links: [],
    },
    {
      id: 11,
      parent: 1,
      text: '1.1 Requirements',
      start: new Date('2018-10-21T09:24:24.319Z'),
      end: new Date('2018-11-22T01:01:08.938Z'),
      percent: 0.29,
      links: [
        {
          target: 12,
          type: 'FS',
        },
      ],
    },
    {
      id: 12,
      parent: 1,
      text: '1.2 Design',
      start: new Date('2018-11-05T09:24:24.319Z'),
      end: new Date('2018-12-12T09:32:51.245Z'),
      percent: 0.78,
    },
  ]

  tasks = []
  loaded = false
  
  async processTasks() {
    try {
      const tasks = await getTasks()
      interface taskInterface {
        id: number
        start: string
        end: string
        duration: number
        type: string
        dependentOn: number[]
        percent: number
        user: string
        style: Object
        label: string
        parentId: number
      }
      console.log(tasks)
      tasks.every(element => {
        // console.log(element, Object.keys(element))
        let task: taskInterface = {}

        task["label"] = "task from the table"
        task["user"] = "origin dev"
        task["percent"] = 100

        if (Object.keys(element).includes('_id')) task['id'] = element['_id']

        if (Object.keys(element).includes('start'))
          task['start'] = new Date(element['start'])

        if (Object.keys(element).includes('duration'))
          task['end'] = new Date(Date.parse(element['start']) + element['duration'] * 3600)

        if (!task['duration']) {
          task['type'] = 'milestone'
          task['duration'] = 100
        } else task['type'] = 'task'

        let dependOn: number[] = []

        if (Object.keys(element).includes('predecessors'))
          element['predecessors'].forEach((predecessor) => {
            dependOn.push(predecessor['n'])
          })
        task['dependentOn'] = dependOn
        console.log(task)
        this.tasks.push(task)
        if (this.tasks.length > 500) return false
        return true
      })
      console.log('returned')
      console.log(this.tasks)
      this.loaded = true
    } catch (err) {
      console.log(err)
    }
  }

  mounted() {
    this.processTasks().then(() => {
      let svgGantt = new SVGGantt('#svg-root', this.tasks, {
      viewMode: 'week',
    })

    let canvasGantt = new CanvasGantt('#canvas-root', this.tasks, {
      viewMode: 'week',
    })

    let strGantt = new StrGantt(this.tasks, {
      viewMode: 'week',
    })
    let body = strGantt.render()
    })

    
  }
}
</script>