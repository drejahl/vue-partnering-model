<template>
  <v-container fluid grid-list-md>
    <v-layout row wrap>
      <div id="app">
        <svg xmlns="http://www.w3.org/2000/svg"
             :width="width+'px'"
             :height="height+'px'"
             @mousemove="drag($event)"
             @mouseup="drop()"
             v-if="bounds.minX">

          <line v-for="link in graph.links"
                :x1="coords[link.source.index].x"
                :y1="coords[link.source.index].y"
                :x2="coords[link.target.index].x"
                :y2="coords[link.target.index].y"
                stroke="black" stroke-width="2"/>

          <circle v-for="(node, i) in graph.nodes"
                  :cx="coords[i].x"
                  :cy="coords[i].y"
                  :r="node.radius" :fill="node.color"
                  stroke="white" stroke-width="1"
                  @mousedown="currentMove = {x: $event.screenX, y: $event.screenY, node: node}"/>

        </svg>
      </div>
    </v-layout>
  </v-container>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'vue-partnering-model',
  components: {
  },
  props: {
    'bmc': Object
  },
  data() {
    return {
      graph: {
        nodes: [],
        links: [],
      },
      width: 400, //Math.max(document.documentElement.clientWidth, window.innerWidth || 0),
      height: 200, //Math.max(document.documentElement.clientHeight, window.innerHeight || 0) - 40,
      padding: 20,
      simulation: null,
      currentMove: null
    }
  },
  watch: {
    bmc: function() {
      this.graph.nodes = this.createNodes();
      this.graph.links = this.createLinks();
      this.startSimulation();
    }
  },
  computed: {
    bounds() {
      return {
        minX: Math.min(...this.graph.nodes.map(n => n.x)),
        maxX: Math.max(...this.graph.nodes.map(n => n.x)),
        minY: Math.min(...this.graph.nodes.map(n => n.y)),
        maxY: Math.max(...this.graph.nodes.map(n => n.y))
      }
    },
    coords() {
      return this.graph.nodes.map(node => {
        return {
          x: this.padding + (node.x - this.bounds.minX) * (this.width - 2*this.padding) / (this.bounds.maxX - this.bounds.minX),
          y: this.padding + (node.y - this.bounds.minY) * (this.height - 2*this.padding) / (this.bounds.maxY - this.bounds.minY)
        }
      })
    }
  },
  created(){
    this.graph.nodes = this.createNodes();
    this.graph.links = this.createLinks();
    this.startSimulation();
  },
  methods: {
    startSimulation: function() {
      this.simulation = d3.forceSimulation(this.graph.nodes)
        .force('charge', d3.forceManyBody().strength(d => -100))
        .force('link', d3.forceLink(this.graph.links))
        .force('x', d3.forceX())
        .force('y', d3.forceY())
    },
    createLinks: function() {
      let links = [];
      let i = 1;
      if ( this.bmc && this.bmc.keyPartners ) {
        this.bmc.keyPartners.forEach( p => { links.push( { source: 0, target: i++ })});
      }
      return links;
    },
    createNodes: function(){
      let nodes = [ { index: 0, color: '#2196F3', radius: 10, x: null, y: null }];
      let i=1;
      if ( this.bmc && this.bmc.keyPartners ) {
        this.bmc.keyPartners.forEach( p => { nodes.push( { index: i++, color:'#009688', radius: 20, x: null, y: null })})
      }
      return nodes;
    },
    drag(e) {
      if (this.currentMove) {
        this.currentMove.node.fx = this.currentMove.node.x - (this.currentMove.x - e.screenX) * (this.bounds.maxX - this.bounds.minX) / (this.width - 2 * this.padding)
        this.currentMove.node.fy = this.currentMove.node.y -(this.currentMove.y - e.screenY) * (this.bounds.maxY - this.bounds.minY) / (this.height - 2 * this.padding)
        this.currentMove.x = e.screenX
        this.currentMove.y = e.screenY
      }
    },
    drop(){
      delete this.currentMove.node.fx
      delete this.currentMove.node.fy
      this.currentMove = null
      this.simulation.alpha(1)
      this.simulation.restart()
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
