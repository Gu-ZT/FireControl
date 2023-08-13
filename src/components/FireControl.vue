<script setup lang="ts">
import {ref} from "vue";

let canFireFlag = ref(true);

function pack(count: number): { fill, sixteen, eight, four, two, one } {
  if (count < 0) canFireFlag.value = flase;
  let out = {
    fill: 0,
    sixteen: 0,
    eight: 0,
    four: 0,
    two: 0,
    one: 0
  }
  out.fill = Math.floor(count / 27);
  count %= 27;
  out.sixteen = Math.floor(count / 16);
  count %= 16;
  out.eight = Math.floor(count / 8);
  count %= 8;
  out.four = Math.floor(count / 4);
  count %= 4;
  out.two = Math.floor(count / 2);
  out.one = Math.floor(count % 2);
  return out;
}

function calculate(radA: number, dist: number, sideT: number, straightT: number, flag: boolean = false)//射程大致计算,转换为TNT数目
{
  let distSTR, tntSTR, needSTR;//既定的正向距离、现有TNT能达到的正向距离与需要多少TNT达到剩余的正向距离
  let needTNTSTR;//正向另外所需的TNT数量
  distSTR = dist * Math.cos(radA);//正向距离
  let distSD, tntSD;//定义偏移量以及所需TNT
  distSD = dist * Math.sin(radA);//侧向距离
  tntSD = distSD / sideT;//20个矿车31m
  tntSTR = tntSD * straightT;//20个矿车77m
  needSTR = distSTR - tntSTR;//确定还要多少TNT
  {
    needTNTSTR = needSTR / straightT;
    needTNTSTR = needTNTSTR / 2;
  }
  return flag ? needTNTSTR : needTNTSTR + tntSD;
}

function toRad(angle: number) {
  let radA;
  radA = angle / 360;
  radA = radA * 2 * 3.1415926;
  if (Math.tan(radA) > 0.5) {
    canFireFlag.value = false;
  }
  return radA;
}

let theories = [
  {id: 0, label: "理论模式", sideT: 1.55, straightT: 3.85},
  {id: 1, label: "实战模式", sideT: 1.01, straightT: 2.5}
]
let theoryRef = ref(theories[0].id);
let posSystems = [
  {id: 0, label: "极坐标系"},
  {id: 1, label: "平面直角坐标系"}
]
let posSystemRef = ref(posSystems[0].id);
let angleRef = ref(0);
let distanceRef = ref(0);
let directions = [
  {id: 0, label: "左"},
  {id: 1, label: "右"}
]
let directionRef = ref(directions[0].id);
let targetPosRef = ref({x: 0, z: 0});
let leftRef = ref({
  fill: 0,
  sixteen: 0,
  eight: 0,
  four: 0,
  two: 0,
  one: 0
});
let rightRef = ref({
  fill: 0,
  sixteen: 0,
  eight: 0,
  four: 0,
  two: 0,
  one: 0
});

function get() {
  canFireFlag.value = true;
  let angle;
  if (posSystemRef.value == 1) {
    if (targetPosRef.value.x <= 0) canFireFlag.value = false;
    distanceRef.value = Math.floor((targetPosRef.value.x ** 2 + targetPosRef.value.z ** 2) ** 0.5);
    // angleRef.value = Math.floor(Math.atan2(targetPosRef.value.x, targetPosRef.value.z) * 180 / Math.PI)
    angle = Math.atan2(targetPosRef.value.z, targetPosRef.value.x);
    directionRef.value = targetPosRef.value.z > 0 ? 0 : 1;
  } else angle = toRad(angleRef.value);
  let distance = distanceRef.value;
  let theory = theories[theoryRef.value];
  let left = calculate(angle, distance, theory.sideT, theory.straightT, true);
  let right = calculate(angle, distance, theory.sideT, theory.straightT);
  if (directionRef.value == 1) {
    let mid = left;
    left = right;
    right = mid;
  }
  if (!canFireFlag) return "打不着的，洗洗睡吧";
  leftRef.value = pack(left);
  rightRef.value = pack(right);
  return {
    left: leftRef.value,
    right: rightRef.value
  }
}

function format(data: { fill, sixteen, eight, four, two, one }) {
  return `【27】\t${data.fill}\n【16】\t${data.sixteen}\n【8】\t${data.eight}\n【4】\t${data.four}\n【2】\t${data.two}\n【1】\t${data.one}`;
}
</script>

<template>
  <div v-if="canFireFlag">
    <el-row>
      <el-col :span="12">
        <el-form label-width="100">
          <el-form-item label="左侧需要："></el-form-item>
          <el-form-item label="【27】">{{ leftRef.fill }}</el-form-item>
          <el-form-item label="【16】">{{ leftRef.sixteen }}</el-form-item>
          <el-form-item label="【8】">{{ leftRef.eight }}</el-form-item>
          <el-form-item label="【4】">{{ leftRef.four }}</el-form-item>
          <el-form-item label="【2】">{{ leftRef.two }}</el-form-item>
          <el-form-item label="【1】">{{ leftRef.one }}</el-form-item>
        </el-form>
      </el-col>
      <el-col :span="12">
        <el-form label-width="100">
          <el-form-item label="右侧需要："></el-form-item>
          <el-form-item label="【27】">{{ rightRef.fill }}</el-form-item>
          <el-form-item label="【16】">{{ rightRef.sixteen }}</el-form-item>
          <el-form-item label="【8】">{{ rightRef.eight }}</el-form-item>
          <el-form-item label="【4】">{{ rightRef.four }}</el-form-item>
          <el-form-item label="【2】">{{ rightRef.two }}</el-form-item>
          <el-form-item label="【1】">{{ rightRef.one }}</el-form-item>
        </el-form>
      </el-col>
    </el-row>
  </div>
  <div v-if="!canFireFlag">
    <el-text type="danger">"打不着的，洗洗睡吧"</el-text>
  </div>
  <el-form label-width="80">
    <el-form-item label="效果选择">
      <el-select v-model="theoryRef" placeholder="效果选择" @change="get">
        <el-option v-for="theory in theories" :key="theory.id" :label="theory.label" :value="theory.id"/>
      </el-select>
    </el-form-item>
    <el-form-item label="敌方坐标">
      <el-select v-model="posSystemRef" placeholder="敌方坐标" @change="get">
        <el-option v-for="posSystem in posSystems" :key="posSystem.id" :label="posSystem.label" :value="posSystem.id"/>
      </el-select>
    </el-form-item>
    <div v-if="posSystemRef == 0">
      <el-form-item label="偏向角度">
        <el-input-number v-model="angleRef" @change="get" @click="get"/>
      </el-form-item>
      <el-form-item label="目标距离">
        <el-input-number v-model="distanceRef" @change="get" @click="get"/>
      </el-form-item>
      <el-form-item label="目标位置">
        <el-select v-model="directionRef" placeholder="目标位置" @change="get">
          <el-option v-for="direction in directions" :key="direction.id" :label="direction.label"
                     :value="direction.id"/>
        </el-select>
      </el-form-item>
    </div>
    <div v-if="posSystemRef == 1">
      <el-form-item label="目标X" @change="get" @click="get">
        <el-input-number v-model="targetPosRef.x"/>
      </el-form-item>
      <el-form-item label="目标Z" @change="get" @click="get">
        <el-input-number v-model="targetPosRef.z"/>
      </el-form-item>
    </div>
  </el-form>
</template>

<style scoped lang="scss">
</style>
