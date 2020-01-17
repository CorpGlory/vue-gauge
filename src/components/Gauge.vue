<template>
  <div>
    <h6 class="gauge-title">{{ title }}</h6>
    <div :id="id"></div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop, Watch } from 'vue-property-decorator';

import * as d3 from 'd3';

const DEFAULT_COLORS = ['green', 'yellow', 'red'];

const DEFAULT_MARGIN = { top: 0, right: 0, bottom: 0, left: 0 };

@Component
export default class Gauge extends Vue {

  @Prop({ required: true })
  id!: string;

  @Prop({ required: true, default: () => DEFAULT_COLORS })
  colors!: string[];

  @Prop({ required: true })
  value!: number;

  @Prop({ required: true })
  stops!: number[];

  @Prop({ required: true })
  maxValue!: number;

  @Prop({ required: false })
  title!: string;

  @Prop({ required: false, default: 200 })
  width!: number;

  @Prop({ required: false, default: 110 })
  height!: number;

  @Prop({ required: false, default: () => DEFAULT_MARGIN })
  margin!: { top: number, right: number, bottom: number, left: number };

  @Prop({ required: false })
  innerRadius!: number;

  @Prop({ required: false })
  outerRadius!: number;

  @Watch('value')
  onValueChange() {
    if(this.value !== undefined) {
      this.renderLine();
    }
  }

  svg: any = undefined;
  gaugeCenter: string = "";

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

  get gaugeOuterRadius(): number {
    return this.outerRadius || this.width / 2 - this.margin.left; 
  }

  get gaugeInnerRadius(): number {
    return this.innerRadius || this.gaugeOuterRadius * 0.625; 
  }

  renderLine(): void {
    let scale = d3.scaleLinear().domain([0, this.maxValue]).range([0,180]);
    this.svg.selectAll('.needle').data([this.value])
      .transition()
      .ease(d3.easeElasticOut)
      .duration(1000)
      .attr('transform', (d: number) => {
        return this.gaugeCenter + 'rotate(' + scale(d) + ')'
      });
  }

  mounted() {
    this.svg = d3.select(`#${this.id}`)
      .append('svg')
      .attr('width', this.width)
      .attr('height', this.height);

    this.gaugeCenter = `translate(${this.width / 2},${this.height - 10})`;

    let arc = d3.arc()
      .innerRadius(this.gaugeInnerRadius)
      .outerRadius(this.gaugeOuterRadius)
      .padAngle(0);

    let pie = d3.pie()
      .startAngle( (-1*Math.PI) / 2 )
      .endAngle( Math.PI / 2 )

    let arcs = pie(this.valueRange);
    let background = this.svg.selectAll('path')
    		.data(arcs)
    		.enter()
    		.append('path')
        .style('fill', (d: object, i: number) => {
          return this.colors[i];
        })
        .attr('d', arc)
        .attr( 'transform', this.gaugeCenter )

    let needle = this.svg.selectAll('.needle')
      .data([0])
      .enter()
      .append('line')
      .attr('x1', 0)
      .attr('x2', -1 * this.gaugeOuterRadius)
      .attr('y1', 0)
      .attr('y2', 0)
      .classed('needle', true)
      .style('stroke', 'black')
      .attr('transform', (d: number) => {
        return this.gaugeCenter + 'rotate(' + d + ')'
      });

    this.renderLine();
  }
}
</script>

