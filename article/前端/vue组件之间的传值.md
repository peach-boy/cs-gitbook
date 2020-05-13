# vue组件之间的传值

## 一、父组件=>子组件
子组件使用props自定义属性
```
export default {
  name: 'SampleChooseDialog',
  props: {
    cardFactoryId: {
      type: Number,
      default: null
    }
  }
} 
```
父组件 使用:card-factory-id传值
```
 <sample-choose-dialog
      :card-factory-id="currentFactoryId"/>
```

## 二、子组件=>父组件
子组件定义事件
```
      this.$emit('confirm', this.selected)
```
父组件将事件绑定方法获取传值
```
  <sample-choose-dialog
      @confirm="onDialogConfirm"/>

   onDialogConfirm(val) {
    //获取子组件的传值
     console.log(val)
    }

```

## 三、兄弟组件之间的传值
todo
