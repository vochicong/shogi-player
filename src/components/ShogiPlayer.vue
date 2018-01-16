<template>
<div class="shogi-player">
  <p>{{mediator.turn_now}}手目</p>
  <div class="board_container board_turn" :class="{enable: board_turn}">
    <PieceStand :location_key="'white'"/>
    <div class="flex_item board_wrap">
      <div class="navi_surface previous" @click="move_to_previous"></div>
      <div class="navi_surface next" @click="move_to_next"></div>
      <table>
        <tr v-for="y in mediator.dimension">
          <template v-for="x in mediator.dimension">
            <td :class="mediator.cell_class(x -1, y - 1)">
              {{mediator.cell_view(x - 1, y - 1)}}
            </td>
          </template>
        </tr>
      </table>
    </div>
    <PieceStand :location_key="'black'"/>
  </div>
  <p>
    <div class="controller">
      <div class="--btn-group">
        <button ref="first"    class="btn btn-default first"      @click="move_to_first">|◀</button>
        <button ref="previous" class="btn btn-default previous"   @click="move_to_previous">◀</button>
        <button ref="next"     class="btn btn-default next"       @click="move_to_next">▶</button>
        <button ref="last"     class="btn btn-default last"       @click="move_to_last">▶|</button>
        <button class="btn btn-default board_turn" @click="board_turn = !board_turn">&#x21BB;</button>
      </div>
    </div>
  </p>
  <p>
    <input type="range" v-model.number="current_turn" :min="mediator.sfen_parser.turn_min" :max="mediator.sfen_parser.turn_max" />
  </p>
  <p>
    <input type="text" :value="mediator.to_sfen" class="form-control" readonly="readonly"/>
  </p>
  <template v-if="env !== 'production'">
    {{mediator.hold_pieces}}
  </template>
</div>
</template>

<script>
import { Mediator } from "../mediator"
import PieceStand from "./piece_stand"
import _ from "lodash"

// for vue template
// import Vue from 'vue'
// import lodash from "lodash"
// Object.defineProperty(Vue.prototype, '$lodash', {value: lodash})

/* eslint-disable no-new */
export default {
  name: 'ShogiPlayer',

  props: [
    "kifu_body",
    "turn_start",
    "keyboard_operation_flag",
    "location_hash_embed_turn",
  ],

  components: {
    PieceStand,
  },

  data() {
    return {
      current_turn: 0,
      mediator: null,
      board_turn: false,
      env: process.env.NODE_ENV,
    }
  },

  created() {
    this.current_turn = this.turn_start || 0
    this.mediator_update()
    document.addEventListener("keydown", this.keyboard_operation)
  },

  watch: {
    current_turn: function () {
      this.mediator_update()
    },
    kifu_body: function () {
      this.mediator_update()
    },
  },

  methods: {
    keyboard_operation: function (e) {
      if (this.env !== "production") {
        console.log(document.activeElement)
        console.log(e.shiftKey, e.ctrlKey, e.altKey, e.metaKey)
        console.log("e", e)
        console.log("key", e.key)
        console.log("code", e.code)
      }

      if (!this.keyboard_operation_flag) {
        return
      }

      const dom = document.activeElement
      const controllers = [this.$refs.first, this.$refs.previous, this.$refs.next, this.$refs.last] // FIXME: 指定DOMの下にあるか？の方法がわかればもっと簡潔になる
      if (!(dom === undefined || dom.tagName === "BODY" || _.includes(controllers, dom))) {
        return
      }

      if (e.code === "Backspace" || e.code === "ArrowUp" || e.code === "ArrowLeft" || e.key === "k" || e.key === "p" || e.key === "b") {
        this.move_to_previous()
        e.preventDefault()
      }

      if (e.code === "Space" || e.code === "Enter" || e.code === "ArrowDown" || e.code === "ArrowRight" || e.key === "j" || e.key === "n" || e.key === "f") {
        this.move_to_next()
        e.preventDefault()
      }

      // let gap = null
      // let force_value = null
      //
      // if (e.code === "Space" || e.code === "Enter" || e.code === "ArrowDown" || e.code === "ArrowRight" || e.key === "j" || e.key === "n" || e.key === "f") {
      //   gap = 1
      // }
      // if (e.code === "Backspace" || e.code === "ArrowUp" || e.code === "ArrowLeft" || e.key === "k" || e.key === "p" || e.key === "b") {
      //   gap = -1
      // }
      // if (e.key === "PageUp") {
      //   gap = -10
      // }
      // if (e.key === "PageDown") {
      //   gap = 10
      // }
      // if (e.shiftKey || e.ctrlKey || e.altKey || e.metaKey) {
      //   if (gap) {
      //     gap *= 10
      //   }
      // }
      // if (e.key === "[" || e.key === "Home" || e.code === "Escape") {
      //   force_value = this.mediator.sfen_parser.turn_min
      // }
      // if (e.key === "]" || e.key === "End") {
      //   force_value = this.mediator.sfen_parser.turn_max
      // }
      //
      // let v = this.current_turn
      // if (gap !== null) {
      //   v += gap
      // }
      // if (force_value !== null) {
      //   v = force_value
      // }
      // if (v < this.mediator.sfen_parser.turn_min) {
      //   v = this.mediator.sfen_parser.turn_min
      // }
      // if (this.mediator.sfen_parser.turn_max < v) {
      //   v = this.mediator.sfen_parser.turn_max
      // }
      // this.current_turn = v
      //
      // if (gap !== null || force_value !== null) {
      //   e.preventDefault()
      // }
    },

    mediator_update() {
      this.mediator = new Mediator()
      this.mediator.kifu_body = this.kifu_body || "position startpos"
      this.mediator.current_turn = this.current_turn
      this.mediator.run()
      this.current_turn = this.mediator.turn_now
      if (this.location_hash_embed_turn) {
        document.location.hash = this.current_turn
      }
    },

    move_to_previous() {
      if (this.current_turn > this.mediator.sfen_parser.turn_min) {
        this.current_turn--
      }
      this.$refs.previous.focus()
    },

    move_to_next() {
      if (this.current_turn < this.mediator.sfen_parser.turn_max) {
        this.current_turn++
      }
      this.$refs.next.focus()
    },

    move_to_first() {
      this.current_turn = this.mediator.sfen_parser.turn_min
      this.$refs.first.focus()
    },

    move_to_last() {
      this.current_turn = this.mediator.sfen_parser.turn_max
      this.$refs.last.focus()
    },
  },
}
</script>

<style scoped lang="sass">
  @import "ShogiPlayer"
</style>