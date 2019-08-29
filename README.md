# Carousel

写一个轮子，可复用，有动画效果的轮播效果。

## 实现基本功能的轮播效果

#### 核心功能实现

1. 所有图片层叠在一起（position:absolute）.
2. 然后当点击小圆点，得到小圆点位置（index）,设置小圆点集合的样式（添加删除 class 为 active）
3. 通过获取到的 index 设置图片的 z-index,把要展示的页面放在最上层

**遇到的问题**：比如当小圆点到达最左边的时候，需要从 index=0 跳到 index=arr.length-1，所以就需要对有以下处理:

```javascript
  //用类似于Vue里的计算属性来获取当前(fromIndex)index
  get index() {
    return this.dots.indexOf(this.root.querySelector('.dots .active'))
  }

  //   上一页目标(toIndex)index
  get preIndex() {
    return (this.index-1+this.dots.length) % this.dots.length;
  }

  //   下一页目标(toIndex)index
  get nextIndex() {
    return (this.index+1) % this.dots.length;
  }

  ...
  // 在下面调用
  this.pre.onclick = e => {
      this.showPage(this.preIndex, this.index)
      this.setDot(this.preIndex)

```

#### 函数封装

由于代码重复性太高，需要将重复代码封装成函数来使用。

1. 将设置小圆点 class,封装为 setDots()函数
2. 将设置图片 z-index,封装为 setPages()函数

```javascript
//   改变导航小点的class
function setDots(index) {
  $$(".carousel .dots span").forEach(dot => dot.classList.remove("active"))
  $$(".carousel .dots span")[index].classList.add("active")
}
function setPages(index) {
  $$(".carousel .panels a").forEach(panel => (panel.style.zIndex = 1))
  $$(".carousel .panels a")[index].style.zIndex = 10
}
```

附上实现了基本功能的轮播图：<a href="http://js.jirengu.com/rutuy/51/edit">基本轮播图效果</a>

## 封装成简单的轮播组件

为了使轮播可复用，我将封装成简单的轮播组件，使之变成面向对象的组件化的方式。从而可以满足页面上有多个轮播的需求。

- 用类封装
- 目标 index 的获取用函数封装

#### 封装一个 Carousel 类

```javascript
class Carousel {
  constructor(root) {
    //      注意：类数组对象转换成数组
    this.root = root
    this.dotsCt = root.querySelector(".dots")
    this.dots = Array.from(root.querySelectorAll(".dots > span"))
    this.panels = Array.from(root.querySelectorAll(".panels > a"))
    this.pre = root.querySelector(".action .pre")
    this.next = root.querySelector(".action .next")

    this.bind()
  }

  bind() {
    // 当点击小圆点时
    this.dotsCt.onclick = e => {
      if (e.target.tagName !== "SPAN") return
      let index = this.dots.indexOf(e.target)

      //   改变导航小点的class
      this.setDots(index)
      //   改变图片的index
      this.setPanels(index)
    }

    // 当点击上一页时
    this.pre.onclick = e => {
      let index = this.dots.indexOf(this.root.querySelector(".dots .active"))
      let firstIndex = this.dots.length - 1
      index = index == 0 ? firstIndex : index - 1

      //   改变导航小点的class
      this.setDots(index)
      //   改变图片的index
      this.setPanels(index)
    }

    // 当点击下一页时
    this.next.onclick = e => {
      let index = this.dots.indexOf(this.root.querySelector(".dots .active"))
      let lastIndex = this.dots.length - 1
      index = index == lastIndex ? 0 : index + 1
      console.log("sd")
      //   改变导航小点的class
      this.setDots(index)
      //   改变图片的index
      this.setPanels(index)
    }
  }

  //   改变导航小点的class
  setDots(index) {
    this.dots.forEach(dot => dot.classList.remove("active"))
    this.dots[index].classList.add("active")
  }

  //   改变图片的index
  setPanels(index) {
    this.panels.forEach(panel => (panel.style.zIndex = 1))
    this.panels[index].style.zIndex = 10
  }
}

document
  .querySelectorAll(".carousel")
  .forEach(carousel => new Carousel(carousel))
```

**遇到的问题**：想在函数 bind()里的闭包使用 this.dots 变量时，不对，原因是这个 this 没有正确获取到，解决方法是：闭包使用箭头函数，从而继承上一级的 this.

```javascript
class Carousel {
  constructor(root) {
    this.root=root;
    this.dots=Array.from(root.querySelectorAll('.dots > span'));
    ...
    ...
    this.bind()
  }

    bind() {
    this.dotsCt.onclick=(e) => {
        if(e.target.tagName!=='SPAN') return
        let index=this.dots.indexOf(e.target) // 此处的this继承于上一级的this

        this.setDots(index)
        this.setPanels(index)
    }
}
```

**遇到的问题**：将上一页下一页的获取目标 index 的函数分别封装后，再用 setDots() setPanels()调用时出错，因为首先调用了 setDots() ，使得下面的小圆点已经改变了，目标 index 也就改变了，此时再调用 setPanels() 获取到的是改变后的 index,index 获取的不准确了。解决方法是：先调用 setPanels(),再调用 setDots().

```javascript
  //用类似于Vue里的计算属性来获取要切换到的index
  get index() {
    return this.dots.indexOf(this.root.querySelector('.dots .active'))
  }

//   上一页index
  get preIndex() {
    return (this.index-1+this.dots.length) % this.dots.length;
  }
//   下一页index
  get nextIndex() {
    return (this.index+1) % this.dots.length;
  }

          // 当点击上一页时
        this.pre.onclick=(e) => {
          this.setPanels(this.preIndex)//先调用 setPanels()
          this.setDots(this.preIndex)//再调用 setDots().
        }

```

附上实现了封装的组件化的轮播效果：<a href="http://js.jirengu.com/luceq/59/edit">用类封装后的轮播效果</a>

## 实现切换的动画效果

为了使我写的轮播不只是单纯的图片替换，所以决定对切换效果做丰富的扩充。
这种效果还可以用在 Tab 的切换效果

写一个 Animation 对象，里面存储动画函数:

```javascript
class Animation={
  fade(){
   ...
  },
  slide(){
   ...
  },
   ...
}
```

**fade 动画**：

```javascript
// 改变fromNode和toNode的opacity
 fade(step = 0.04) {
    return function (fromNode, toNode, onFinish) {
      console.log(fromNode, toNode)
      let opacityOffset1 = 1
      let opacityOffset2 = 0
      fromNode.style.zIndex = 10
      toNode.style.zIndex = 9

      function fromNodeAnimation() {
        if (opacityOffset1 > 0) {
          opacityOffset1 -= step
          fromNode.style.opacity = opacityOffset1
          requestAnimationFrame(fromNodeAnimation)
        } else {
          fromNode.style.opacity = 0
        }
      }

      function toNodeAnimation() {
        if (opacityOffset2 < 1) {
          opacityOffset2 += step
          toNode.style.opacity = opacityOffset2
          requestAnimationFrame(toNodeAnimation)
        } else {
          fromNode.style.opacity = 1
          toNode.style.opacity = 1
          onFinish()
        }

      }

      fromNodeAnimation()
      toNodeAnimation()
    }
  },
```

**slide 动画**：

```javascript
// 改变fromNode和toNode的left
 slide(step = 10) {
    return function (fromNode, toNode, onFinish) {
      fromNode.style.zIndex = 10
      toNode.style.zIndex = 10

      let width = parseInt(getComputedStyle(fromNode).width)
      let offsetX = width //要水平移动的举例
      let offset1 = 0 //第一个元素已经移动的举例
      let offset2 = 0 //第二个元素已经移动的举例
      //let step = 10   //每次移动的举例

      toNode.style.left = width + 'px'

      function fromNodeAnimation() {
        if (offset1 < offsetX) {
          fromNode.style.left = parseInt(getComputedStyle(fromNode).left) - step + 'px'
          offset1 += step
          requestAnimationFrame(fromNodeAnimation)
        }
      }

      function toNodeAnimation() {
        if (offset2 < offsetX) {
          toNode.style.left = parseInt(getComputedStyle(toNode).left) - step + 'px'
          offset2 += step
          requestAnimationFrame(toNodeAnimation)
        } else {
          onFinish()
          fromNode.style.left = 0
          toNode.style.left = 0
        }
      }

      fromNodeAnimation()
      toNodeAnimation()
    }

  },
```

**遇到的问题**：再用 slide 动画的时候，如果点击下一页很快，就会出现多个图片的闪烁，用户体验不好，解决方法：使用 css 实现切换

以下是用 css 来做的切换动画，相比于 js 的方法，css 方法简单易懂。

**cssSlide 动画**：

```javascript
// 改变fromNode和toNode的left,用transformX平移
 cssSlide(during = .3) {
    const css = (node, styles) => Object.entries(styles).forEach(([key, value]) => node.style[key] = value)

    return function (fromNode, toNode, onFinish) {
      let width = parseInt(getComputedStyle(fromNode).width)
      let fromNodeIndex = Array.from(fromNode.parentElement.children).indexOf(fromNode)
      let toNodeIndex = Array.from(toNode.parentElement.children).indexOf(toNode)

      css(fromNode, {
        zIndex: 10,
        transition: `transform ${during}s`,
        transform: `translateX(${fromNodeIndex < toNodeIndex ? '-' : ''}100%)`
      })
      css(toNode, {
        zIndex: 10,
        left: `${fromNodeIndex < toNodeIndex ? '' : '-'}${width}px`,
        transition: `transform ${during}s`,
        transform: `translateX(${fromNodeIndex < toNodeIndex ? '-' : ''}100%)`
      })

      setTimeout(() => {
        css(fromNode, {
          transition: 'none',
          transform: `translateX(0)`
        })

        css(toNode, {
          left: 0,
          transition: 'none',
          transform: `translateX(0)`
        })

        onFinish()
      }, during * 1000)

    }
  },

```

**cssZoom 动画**：

```javascript
// 改变fromNode的scale,opacity
 cssZoom(during= .3) {
        const css = (node, styles) => Object.entries(styles).forEach(([key, value]) => node.style[key] = value);

    return function(fromNode,toNode,onFinish) {
      css(fromNode,{
        zIndex:10,
        transition:`all ${during}s`,
        transform:`scale(5)`,
        opacity:0
      })

      css(toNode,{
        zIndex:9
      })

      setTimeout(()=>{
        css(fromNode,{
          transition:'none',
          transform:`none`,
          opacity:1
        })
        onFinish()
      },during*1000)
    }
  }
```

附上 有动画的轮播效果的 demo 链接：<a href="http://js.jirengu.com/vehax/138/edit">有动画的轮播效果</a>
