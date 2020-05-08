<template>
  <div>
    <h6 class="gauge-title" v-bind:style="titlePosition">{{ title }}</h6>
    <div :id="id"></div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, Prop, Watch } from 'vue-property-decorator';

import * as d3 from 'd3';

const DEFAULT_COLORS = ['green', 'yellow', 'red'];

const DEFAULT_MARGIN = { top: 0, right: 0, bottom: 20, left: 0 };

@Component
export default class Gauge extends Vue {

  @Prop({ required: true })
  id!: string;

  @Prop({ required: true, default: () => DEFAULT_COLORS })
  colors!: string[];

  @Prop({ required: false })
  value!: number | undefined;

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

  @Prop({ required: false, default: () => DEFAULT_MARGIN })
  marginTitle!: { top: number, right: number, bottom: number, left: number };

  @Prop({ required: false })
  innerRadius!: number;

  @Prop({ required: false })
  outerRadius!: number;

  @Prop({ required: false, default: false })
  displayLabel!: boolean;

  @Prop({ required: false })
  unit!: string;

  @Watch('value')
  onValueChange() {
    this.renderLine();
    if(this.value !== undefined) {
      this.updateValueText();
    } else {
      this.renderNoDataText();
    }
  }

  svg: any = undefined;
  gaugeCenter: string = "";
  minValue = 0;
  text: any = undefined;

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

  get titlePosition(): any {
    return {
      margin:
        `${this.marginTitle.top}px
        ${this.marginTitle.right}px
        ${this.marginTitle.bottom}px
        ${this.marginTitle.left}px`
    };
  }

  get gaugeValue(): number {
    if(this.value === undefined) {
      return 0;
    }
    return this.value;
  }

  renderLine(): void {
    let scale = d3.scaleLinear()
      .domain([0, this.maxValue])
      .range([0,180])
      .clamp(true);

    this.svg.selectAll('.needle').data([this.gaugeValue])
      .transition()
      .ease(d3.easeElasticOut)
      .duration(1000)
      .attr('transform', (d: number) => {
        return this.gaugeCenter + 'rotate(' + scale(d) + ')'
      });
  }

  renderValueText(): void {
    this.text = this.svg.selectAll('.value-text')
      .data([0])
      .enter()
      .append('text')
      .attr('x', 0)
      .attr('y', -5)
      .text(this.gaugeValue.toFixed(2)+" "+this.unit)
      .classed('value-text', true)
      .attr('font-family', 'Poppins, sans-serif')
      .attr('font-size', '13px')
      .attr('transform', this.gaugeCenter)
      .attr('text-anchor', 'middle')
      .attr('alignment-baseline', 'central')
      .style('font-weight', 'bold');
  }

  updateValueText(): void {
    this.text
      .text(this.gaugeValue.toFixed(2)+" "+this.unit)
  }

  renderNoDataText(): void {
    this.text
      .text('No Data');
  }

  renderExtremumValuesText(): void {
    this.svg.selectAll('.min-value-text')
      .data([0])
      .enter()
      .append('text')
      .attr('x', -this.gaugeOuterRadius)
      .attr('y', this.margin.bottom)
      .text(this.minValue)
      .classed('min-value-text', true)
      .attr('font-family', 'Poppins, sans-serif')
      .attr('font-size', '14px')
      .attr('transform', this.gaugeCenter);

    this.svg.selectAll('.max-value-text')
      .data([0])
      .enter()
      .append('text')
      .attr('x', this.gaugeInnerRadius)
      .attr('y', this.margin.bottom)
      .attr('width', 40)
      .text(this.maxValue)
      .classed('max-value-text', true)
      .attr('font-family', 'Poppins, sans-serif')
      .attr('font-size', '14px')
      .attr('transform', this.gaugeCenter);
  }

  renderUnitValue() {
    this.svg.selectAll('.unit-value-text')
      .data([0])
      .enter()
      .append('text')
      .attr('x', 5)
      .attr('y', 0)
      .text(this.unit)
      .classed('unit-value-text', true)
      .attr('font-family', 'Poppins, sans-serif')
      .attr('font-size', '12px')
      .attr('transform', this.gaugeCenter);
  }

  mounted() {
    this.svg = d3.select(`#${this.id}`)
      .append('svg')
      .attr('width', this.width)
      .attr('height', this.height);

    this.gaugeCenter = `translate(${this.width / 2},${this.height - this.margin.bottom})`;

    let arc = d3.arc()
      .innerRadius(this.gaugeInnerRadius)
      .outerRadius(this.gaugeOuterRadius)
      .padAngle(0);

    let pie = d3.pie()
      .sort(null)
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
      .attr('x1', -1 * this.gaugeInnerRadius)
      .attr('x2', -1 * this.gaugeOuterRadius)
      .attr('y1', 0)
      .attr('y2', 0)
      .classed('needle', true)
      .attr('stroke', 'black')
      .attr('stroke-width', '2px')
      .attr('transform', (d: number) => {
        return this.gaugeCenter + 'rotate(' + d + ')'
      });

    this.renderLine();
    this.renderValueText();
    this.renderExtremumValuesText();
  }
}
</script>
