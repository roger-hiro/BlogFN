## 前言

某天在逛社区时看到一帖子：
> [react-dynamic-charts — A React Library for Visualizing Dynamic Data](https://medium.com/outbrain-engineering/react-dynamic-charts-a-react-library-for-visualizing-dynamic-data-9b299a107e06)

![](https://user-gold-cdn.xitu.io/2019/8/16/16c99d64d86df1bb?w=682&h=553&f=png&s=112508)
这是一个国外大佬在其公司峰会的代码竞赛中写的一个库：`react-dynamic-charts`，用于根据动态数据创建动态图表可视化。
![](https://user-gold-cdn.xitu.io/2019/8/16/16c992c676eb96f9?w=1042&h=538&f=gif&s=1560535)
它的设计非常灵活，允许你控制内部的每个元素和事件。使用方法也非常简单，其源码也是非常精炼，值得学习。

但因其提供了不少`API`,不利于理解源码。所以以下实现有所精简：

## 1. 准备通用工具函数

![](https://user-gold-cdn.xitu.io/2019/8/16/16c99f63294accae?w=221&h=180&f=png&s=63791)
### 1. `getRandomColor`：随机颜色
```
const getRandomColor = () => {
  const letters = '0123456789ABCDEF';
  let color = '#';
  for (let i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)]
  }
  return color;
};
```

### 2. `translateY`：填充Y轴偏移量

```
const translateY = (value) => {
  return `translateY(${value}px)`;
}
```

## 2. 使用`useState Hook`声明状态变量
我们开始编写组件`DynamicBarChart`
```
const DynamicBarChart = (props) =>  {
  const [dataQueue, setDataQueue] = useState([]);
  const [activeItemIdx, setActiveItemIdx] = useState(0);
  const [highestValue, setHighestValue] = useState(0);
  const [currentValues, setCurrentValues] = useState({});
  const [firstRun, setFirstRun] = useState(false);
  // 其它代码...
  }
```
### 1. `useState`的简单理解:
```
const [属性, 操作属性的方法] = useState(默认值);
```

### 2. 变量解析

* `dataQueue`：当前操作的原始数据数组
* `activeItemIdx`: 第几“帧”
* `highestValue`: “榜首”的数据值
* `currentValues`: 经过处理后用于渲染的数据数组
  ![](https://user-gold-cdn.xitu.io/2019/8/16/16c99a64095a59c8?w=1606&h=304&f=png&s=186548)
* `firstRun`: 第一次动态渲染时间

## 3. 内部操作方法和对应`useEffect`
请配合注释食用
```
// 动态跑起来～
function start () {
  if (activeItemIdx > 1) {
    return;
  }
  nextStep(true);
}
// 对下一帧数据进行处理
function setNextValues () {
  // 没有帧数时（即已结束），停止渲染
  if (!dataQueue[activeItemIdx]) {
    iterationTimeoutHolder = null;
    return;
  }
  //  每一帧的数据数组
  const roundData = dataQueue[activeItemIdx].values;
  const nextValues = {};
  let highestValue = 0;
  //  处理数据，用作最后渲染（各种样式，颜色）
  roundData.map((c) => {
    nextValues[c.id] = {
      ...c,
      color: c.color || (currentValues[c.id] || {}).color || getRandomColor()
    };

    if (Math.abs(c.value) > highestValue) {
      highestValue = Math.abs(c.value);
    }

    return c;
  });

  // 属性的操作，触发useEffect
  setCurrentValues(nextValues);
  setHighestValue(highestValue);
  setActiveItemIdx(activeItemIdx + 1);
}
// 触发下一步，循环
function nextStep (firstRun = false) {
  setFirstRun(firstRun);
  setNextValues();
}
 ```
 
对应`useEffect`:
```
// 取原始数据
useEffect(() => {
  setDataQueue(props.data);
}, []);
// 触发动态
useEffect(() => {
  start();
}, [dataQueue]);
// 设触发动态间隔
useEffect(() => {
  iterationTimeoutHolder = window.setTimeout(nextStep, 1000);
  return () => {
    if (iterationTimeoutHolder) {
      window.clearTimeout(iterationTimeoutHolder);
    }
  };
}, [activeItemIdx]);
```
`useEffect`示例：

```
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // 仅在 count 更改时更新
```
**为什么要在 `effect` 中返回一个函数？**

这是 `effect` 可选的清除机制。每个 `effect` 都可以返回一个清除函数。如此可以将添加和移除订阅的逻辑放在一起。

## 4. 整理用于渲染`Dom`的数据
```
const keys = Object.keys(currentValues);
const { barGapSize, barHeight, showTitle } = props;
const maxValue = highestValue / 0.85;
const sortedCurrentValues = keys.sort((a, b) => currentValues[b].value - currentValues[a].value);
const currentItem = dataQueue[activeItemIdx - 1] || {};
```

* `keys`: 每组数据的索引
* `maxValue`: 图表最大宽度
* `sortedCurrentValues`: 对每组数据进行排序，该项影响动态渲染。
* `currentItem`: 每组的原始数据

## 5. 开始渲染...

大致的逻辑就是：
1. 根据不同`Props`,循环排列后的数据：`sortedCurrentValues`
2. 计算宽度，返回每项的`label`、`bar`、`value`
3. 根据计算好的高度，触发`transform`。

```
<div className="live-chart">
{
<React.Fragment>
  {
    showTitle &&
    <h1>{currentItem.name}</h1>
  }
  <section className="chart">
    <div className="chart-bars" style={{ height: (barHeight + barGapSize) * keys.length }}>
      {
        sortedCurrentValues.map((key, idx) => {
          const currentValueData = currentValues[key];
          const value = currentValueData.value
          let width = Math.abs((value / maxValue * 100));
          let widthStr;
          if (isNaN(width) || !width) {
            widthStr = '1px';
          } else {
            widthStr = `${width}%`;
          }

          return (
            <div className={`bar-wrapper`} style={{ transform: translateY((barHeight + barGapSize) * idx), transitionDuration: 200 / 1000 }} key={`bar_${key}`}>
              <label>
                {
                  !currentValueData.label
                    ? key
                    : currentValueData.label
                }
              </label>
              <div className="bar" style={{ height: barHeight, width: widthStr, background: typeof currentValueData.color === 'string' ? currentValueData.color : `linear-gradient(to right, ${currentValueData.color.join(',')})` }} />
              <span className="value" style={{ color: typeof currentValueData.color === 'string' ? currentValueData.color : currentValueData.color[0] }}>{currentValueData.value}</span>
            </div>
          );
        })
      }
    </div>
  </section>
</React.Fragment>
}
</div>
```
## 6. 定义常规`propTypes`和`defaultProps`:
```
DynamicBarChart.propTypes = {
  showTitle: PropTypes.bool,
  iterationTimeout: PropTypes.number,
  data: PropTypes.array,
  startRunningTimeout: PropTypes.number,
  barHeight: PropTypes.number,
  barGapSize: PropTypes.number,
  baseline: PropTypes.number,
};

DynamicBarChart.defaultProps = {
  showTitle: true,
  iterationTimeout: 200,
  data: [],
  startRunningTimeout: 0,
  barHeight: 50,
  barGapSize: 20,
  baseline: null,
};

export {
  DynamicBarChart
};

```

## 7. 如何使用
```
import React, { Component } from "react";

import { DynamicBarChart } from "./DynamicBarChart";

import helpers from "./helpers";
import mocks from "./mocks";

import "react-dynamic-charts/dist/index.css";

export default class App extends Component {
  render() {
    return (
      <DynamicBarChart
            barGapSize={10}
            data={helpers.generateData(100, mocks.defaultChart, {
              prefix: "Iteration"
            })}
            iterationTimeout={100}
            showTitle={true}
            startRunningTimeout={2500}
          />
      )
  }
}
```

### 1. 批量生成`Mock`数据

![](https://user-gold-cdn.xitu.io/2019/8/16/16c99e0f50cf0a78?w=1246&h=586&f=png&s=119640)
 `helpers.js`:
```
function getRandomNumber(min, max) {
  return Math.floor(Math.random() * (max - min + 1) + min);
};

function generateData(iterations = 100, defaultValues = [], namePrefix = {}, maxJump = 100) {
  const arr = [];
  for (let i = 0; i <= iterations; i++) {
    const values = defaultValues.map((v, idx) => {
      if (i === 0 && typeof v.value === 'number') {
        return v;
      }
      return {
        ...v,
        value: i === 0 ? this.getRandomNumber(1, 1000) : arr[i - 1].values[idx].value + this.getRandomNumber(0, maxJump)
      }
    });
    arr.push({
      name: `${namePrefix.prefix || ''} ${(namePrefix.initialValue || 0) + i}`,
      values
    });
  }
  return arr;
};

export default {
  getRandomNumber,
  generateData
}
```
`mocks.js`:
```
import helpers from './helpers';
const defaultChart = [
  {
    id: 1,
    label: 'Google',
    value: helpers.getRandomNumber(0, 50)
  },
  {
    id: 2,
    label: 'Facebook',
    value: helpers.getRandomNumber(0, 50)
  },
  {
    id: 3,
    label: 'Outbrain',
    value: helpers.getRandomNumber(0, 50)
  },
  {
    id: 4,
    label: 'Apple',
    value: helpers.getRandomNumber(0, 50)
  },
  {
    id: 5,
    label: 'Amazon',
    value: helpers.getRandomNumber(0, 50)
  },
];
export default {
  defaultChart,
}
```

一个乞丐版的动态排行榜可视化就做好喇。
![](https://user-gold-cdn.xitu.io/2019/8/16/16c99de2635e9f8a?w=1628&h=900&f=gif&s=369209)

## 8. 完整代码
```
import React, { useState, useEffect } from 'react';
import PropTypes from 'prop-types';
import './styles.scss';

const getRandomColor = () => {
  const letters = '0123456789ABCDEF';
  let color = '#';
  for (let i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)]
  }
  return color;
};

const translateY = (value) => {
  return `translateY(${value}px)`;
}

const DynamicBarChart = (props) => {
  const [dataQueue, setDataQueue] = useState([]);
  const [activeItemIdx, setActiveItemIdx] = useState(0);
  const [highestValue, setHighestValue] = useState(0);
  const [currentValues, setCurrentValues] = useState({});
  const [firstRun, setFirstRun] = useState(false);
  let iterationTimeoutHolder = null;

  function start () {
    if (activeItemIdx > 1) {
      return;
    }
    nextStep(true);
  }

  function setNextValues () {
    if (!dataQueue[activeItemIdx]) {
      iterationTimeoutHolder = null;
      return;
    }
    
    const roundData = dataQueue[activeItemIdx].values;
    const nextValues = {};
    let highestValue = 0;
    roundData.map((c) => {
      nextValues[c.id] = {
        ...c,
        color: c.color || (currentValues[c.id] || {}).color || getRandomColor()
      };

      if (Math.abs(c.value) > highestValue) {
        highestValue = Math.abs(c.value);
      }

      return c;
    });
    console.table(highestValue);

    setCurrentValues(nextValues);
    setHighestValue(highestValue);
    setActiveItemIdx(activeItemIdx + 1);
  }

  function nextStep (firstRun = false) {
    setFirstRun(firstRun);
    setNextValues();
  }

  useEffect(() => {
    setDataQueue(props.data);
  }, []);

  useEffect(() => {
    start();
  }, [dataQueue]);

  useEffect(() => {
    iterationTimeoutHolder = window.setTimeout(nextStep, 1000);
    return () => {
      if (iterationTimeoutHolder) {
        window.clearTimeout(iterationTimeoutHolder);
      }
    };
  }, [activeItemIdx]);

  const keys = Object.keys(currentValues);
  const { barGapSize, barHeight, showTitle, data } = props;
  console.table('data', data);
  const maxValue = highestValue / 0.85;
  const sortedCurrentValues = keys.sort((a, b) => currentValues[b].value - currentValues[a].value);
  const currentItem = dataQueue[activeItemIdx - 1] || {};

  return (
    <div className="live-chart">
      {
        <React.Fragment>
          {
            showTitle &&
            <h1>{currentItem.name}</h1>
          }
          <section className="chart">
            <div className="chart-bars" style={{ height: (barHeight + barGapSize) * keys.length }}>
              {
                sortedCurrentValues.map((key, idx) => {
                  const currentValueData = currentValues[key];
                  const value = currentValueData.value
                  let width = Math.abs((value / maxValue * 100));
                  let widthStr;
                  if (isNaN(width) || !width) {
                    widthStr = '1px';
                  } else {
                    widthStr = `${width}%`;
                  }

                  return (
                    <div className={`bar-wrapper`} style={{ transform: translateY((barHeight + barGapSize) * idx), transitionDuration: 200 / 1000 }} key={`bar_${key}`}>
                      <label>
                        {
                          !currentValueData.label
                            ? key
                            : currentValueData.label
                        }
                      </label>
                      <div className="bar" style={{ height: barHeight, width: widthStr, background: typeof currentValueData.color === 'string' ? currentValueData.color : `linear-gradient(to right, ${currentValueData.color.join(',')})` }} />
                      <span className="value" style={{ color: typeof currentValueData.color === 'string' ? currentValueData.color : currentValueData.color[0] }}>{currentValueData.value}</span>
                    </div>
                  );
                })
              }
            </div>
          </section>
        </React.Fragment>
      }
    </div>
  );
};

DynamicBarChart.propTypes = {
  showTitle: PropTypes.bool,
  iterationTimeout: PropTypes.number,
  data: PropTypes.array,
  startRunningTimeout: PropTypes.number,
  barHeight: PropTypes.number,
  barGapSize: PropTypes.number,
  baseline: PropTypes.number,
};

DynamicBarChart.defaultProps = {
  showTitle: true,
  iterationTimeout: 200,
  data: [],
  startRunningTimeout: 0,
  barHeight: 50,
  barGapSize: 20,
  baseline: null,
};

export {
  DynamicBarChart
};

```
`styles.scss`：
```
.live-chart {
  width: 100%;
  padding: 20px;
  box-sizing: border-box;
  position: relative;
  text-align: center;
  h1 {
    font-weight: 700;
    font-size: 60px;
    text-transform: uppercase;
    text-align: center;
    padding: 20px 10px;
    margin: 0;
  }

  .chart {
    position: relative;
    margin: 20px auto;
  }

  .chart-bars {
    position: relative;
    width: 100%;
  }
  
  .bar-wrapper {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    position: absolute;
    top: 0;
    left: 0;
    transform: translateY(0);
    transition: transform 0.5s linear;
    padding-left: 200px;
    box-sizing: border-box;
    width: 100%;
    justify-content: flex-start;

    label {
      position: absolute;
      height: 100%;
      width: 200px;
      left: 0;
      padding: 0 10px;
      box-sizing: border-box;
      text-align: right;
      top: 50%;
      transform: translateY(-50%);
      font-size: 16px;
      font-weight: 700;
      display: flex;
      justify-content: flex-end;
      align-items: center;
    }

    .value {
      font-size: 16px;
      font-weight: 700;
      margin-left: 10px;
    }

    .bar {
      width: 0%;
      transition: width 0.5s linear;
    }
  }
}
```
  
原项目地址：[react-dynamic-charts](https://github.com/dsternlicht/react-dynamic-charts)
![](https://user-gold-cdn.xitu.io/2019/8/16/16c99e5cc9af8432?w=966&h=664&f=png&s=667700)


## 结语
一直对实现动态排行榜可视化感兴趣，无奈多数都是基于`D3`或`echarts`实现。
而这个库，不仅脱离图形库，还使用了`React 16`的新特性。也让我彻底理解了`React Hook`的妙用。

![](https://user-gold-cdn.xitu.io/2019/8/16/16c99efa3eb3377e?w=239&h=179&f=png&s=85405)
## ❤️ 看完三件事

如果你觉得这篇内容对你挺有启发，我想邀请你帮我三个小忙：


* 点个「赞」，让更多的人也能看到这篇内容（喜欢不点在看，都是耍流氓 -_-）
* 关注公众号「前端劝退师」，不定期分享原创&精品技术文章。
* 添加微信：huab119，回复：加群。加入前端劝退师公众号交流群。

![](https://user-gold-cdn.xitu.io/2019/8/16/16c99f6a1eb7cd79?w=1080&h=546&f=png&s=336649)
懒得`clone`项目的可以公众号后台回复「**可视化**」，直接拿核心代码，拖进项目用。


![](https://user-gold-cdn.xitu.io/2019/8/16/16c99f2b03900f38?w=243&h=180&f=gif&s=79919)
