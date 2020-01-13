<template>
  <div>
    <h3>Gauge</h3>
    <div id="gauge"></div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop, Watch } from 'vue-property-decorator';

import * as d3 from '@types/d3';

const DEFAULT_COLORS = ['green', 'yellow', 'red'];

@Component
export default class Gauge extends Vue {

  @Prop({ required: true, default: () => DEFAULT_COLORS })
  colors!: string[];

  @Prop({ required: true })
  value!: number;

  @Prop({ required: true })
  stops!: number[];

  @Prop({ required: true })
  maxValue!: number;

  @Prop({ required: false, default: 200 })
  width!: number;

  @Prop({ required: false, default: 100 })
  height!: number;

  svg: any = undefined;
  arc: any = undefined;
  scale: any = undefined;

  get valueRange(): number[] {
    if(this.stops.length < 2) {
      return this.stops;
    }
    let range = [this.stops[0]];
    for(let i = 1; i < this.stops.length; i++) {
      range.push(this.stops[i]-this.stops[i-1]);
    }
    return range;
  }

  renderLine(): void {
    let scale = d3.scaleLinear().domain([0, this.maxValue]).range([0,180]);
    this.svg.selectAll( '.needle' ).data([this.value])
      .transition()
      .ease(d3.easeElasticOut)
      .duration(1000)
      .attr('transform', function(d: any) {
        return 'translate(100,100) rotate(' + scale(d) + ')'
      });
  }

  mounted() {
    this.svg = d3.select('#gauge')
      .append('svg')
      .attr('width', 200)
      .attr('height', 100);

    this.arc = d3.arc()
      .innerRadius(50)
      .outerRadius(80)
      .padAngle(0);

    let scale = d3.scaleLinear().domain([0, this.maxValue]).range([0,180]);

    let pie = d3.pie()
      .startAngle( (-1*Math.PI) / 2 )
      .endAngle( Math.PI / 2 )

    let arcs = pie(this.valueRange);
    let background = this.svg.selectAll('path')
    		.data(arcs)
    		.enter()
    		.append('path')
        .style('fill', function(d: any, i: number){
          return DEFAULT_COLORS[i];
        })
        .attr('d', this.arc)
        .attr( 'transform', 'translate(100,100)' )

    let needle = this.svg.selectAll('.needle')
      .data([0])
      .enter()
      .append('line')
      .attr('x1', 0)
      .attr('x2', -78)
      .attr('y1', 0)
      .attr('y2', 0)
      .classed('needle', true)
      .style('stroke', 'black')
      .attr('transform', function(d: any) {
        return ' translate(100,100) rotate(' + d + ')'
      });
  
    this.svg.selectAll( '.needle' ).data([this.value])
      .transition()
      .ease(d3.easeElasticOut)
      .duration(1000)
      .attr('transform', function(d: any) {
        return 'translate(100,100) rotate(' + scale(d) + ')'
      });
  }
}
</script>

