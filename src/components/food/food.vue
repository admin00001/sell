<template>
  <div class="food" v-show="showFlag" transition="move" v-el:food>
    <div class="food-content">
      <div class="image-header">
        <img :src="food.image">
        <div class="back" @click="hide()">
          <i class="icon-arrow_lift"></i>
        </div>
      </div>
      <div class="content border-1px">
        <div class="name">{{food.name}}</div>
        <div class="desc">
          <span class="count">月售{{food.sellCount}}份</span><span class="rating">好评率{{food.rating}}%</span>
        </div>
        <div class="price">
          <span class="now">&yen;{{food.price}}</span>
          <span class="old" v-show="food.oldPrice">&yen;{{food.oldPrice}}</span>
        </div>
        <div class="cartcontrol-wrapper">
          <cartcontrol :food="food"></cartcontrol>
        </div>
        <div class="buy" v-show="!food.count||food.count===0" @click.stop.prevent="addFirst" transition="fade">
          <span class="desc">加入购物车</span>
        </div>
      </div>
      <split v-show="food.info"></split>
      <div class="info" v-if="food.info">
        <div class="title">商品介绍</div>
        <div class="text">{{food.info}}</div>
      </div>
      <split></split>
      <div class="rating">
        <h1 class="title">商品评价</h1>
        <ratingselect :select-type="selectType" :only-content="onlyContent" :desc="desc"
                      :ratings="food.ratings"></ratingselect>
        <div class="rating-wrapper">
          <ul v-show="food.ratings && food.ratings.length">
            <li v-show="needShow(rating.rateType,rating.text)" class="rating-item border-1px"
                v-for="rating in food.ratings">
              <div class="user">
                <span class="name">{{rating.username}}</span>
                <img class="avatar" width="12" height="12" :src="rating.avatar">
              </div>
              <div class="time">{{rating.rateTime | formatDate}}</div>
              <p class="text-wrapper">
                <span :class="{'icon-thumb_up':rating.rateType===0,'icon-thumb_down':rating.rateType===1}"></span>
                <span class="text">{{rating.text}}</span>
              </p>
            </li>
          </ul>
          <div v-show="!food.ratings || !food.ratings.length" class="no-rating">暂无评价</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  import Vue from 'vue';
  import {formatDate} from 'common/js/date';
  import split from 'components/split/split';
  import ratingselect from 'components/ratingselect/ratingselect';

  const ALL = 2;

  export default{
    props: {
      food: {
        type: Object
      }
    },
    data () {
      return {
        showFlag: false,
        selectType: ALL,
        onlyContent: true,
        desc: {
          all: '全部',
          positive: '推荐',
          negative: '吐槽'
        }
      };
    },
    methods: {
      show() {
        this.showFlag = true;
        this.selectType = ALL;
        this.onlyContent = true;
        this.$nextTick(() => {
          if (!this.scroll) {
            this.scroll = new BScroll(this.$els.food, {
              click: true
            });
          } else {
            this.scroll.refresh();
          }
        });
      },
      hide() {
        this.showFlag = false;
      },
      addFirst(event) {
        if (!event._constructed) {
          return;
        }
        Vue.set(this.food, 'count', 1);
        this.$dispatch('cart.add', event.target);
      },
      needShow(type, text) {
        if (this.onlyContent && !text) {
          return false;
        }
        if (this.selectType === ALL) {
          return true;
        } else {
          return type === this.selectType;
        }
      }
    },
    events: {
      'ratingtype.select'(type) {
        this.selectType = type;
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      },
      'content.toggle'(onlyContent) {
        this.onlyContent = onlyContent;
        this.$nextTick(() => {
          this.scroll.refresh();
        });
      }
    },
    filters: {
      formatDate(time) {
        let date = new Date(time);
        return formatDate(date, 'yyyy-MM-dd hh:mm');
      }
    },
    components: {
      cartcontrol,
      split,
      ratingselect
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"
  .food
    position: fixed
    z-index: 30
    top: 0
    left: 0
    bottom: 48px
    width: 100%
    background: #fff;
    transition: all .5s linear
    &.move-transition
      opacity: 1
    &.move-enter, &.move-leave
      opacity: 0
      left: 100%
    .food-content
      .image-header
        position: relative
        width: 100%
        padding-top: 100%
        img
          position: absolute
          top: 0
          left: 0
          display block
          width: 100%
          height: 100%
        .back
          position: absolute
          top: 10px
          left: 10px
          width: 25px
          height: 25px
          border-radius: 50%
          color: #fff
          background: rgba(7, 17, 27, .3)
          line-height: 25px
          text-align: center
          .icon-arrow_lift
            font-size: 14px
      .content
        padding: 18px 0 0 18px
        background: #fff
        border-1px(rgba(7, 17, 27, .1))
        .name
          height: 14px
          margin-bottom: 8px
          line-height: 14px
          font-weight: 700
          font-size: 14px
          color: rgb(7, 17, 27)
        .desc
          width: 100%
          font-size: 0
          color: rgb(147, 153, 159)
          .count, .rating
            line-height: 10px
            font-size: 10px
          .rating
            margin-left: 12px
        .price
          height: 24px
          margin-top: 18px
          padding-bottom: 18px
          font-weight: 700
          line-height: 24px
          font-size: 0
          .now
            margin-right: 8px
            font-size: 14px
            color: rgb(200, 20, 20)
          .old
            text-decoration: line-through
            font-size: 10px
            color: rgb(147, 153, 159)
        .cartcontrol-wrapper
          position: absolute
          right: 18px
          bottom: 12px
        .buy
          position: absolute
          z-index: 32
          right: 18px
          bottom: 18px
          font-size: 12px
          padding: 0 6px
          border-radius: 12px
          background-color: rgb(0, 160, 220)
          box-sizing: border-box
          text-align: center
          &.fade-transition
            opacity: 1
            transition: all .2s
          &.fade-enter, &.fade-leave
            opacity: 0
          .desc
            line-height: 24px
            font-size: 10px
            color: #fff
      .info
        padding: 18px;
        .title
          margin-bottom: 6px
          line-height: 14px
          font-size: 14px
          color: rgb(7, 17, 27)
        .text
          padding: 0 8px
          line-height: 24px
          font-size: 12px
          color: rgb(147, 153, 159)
      .rating
        padding-top: 18px
        .title
          margin-left: 18px
          line-height: 14px
          font-size: 14px
          color: rgb(7, 17, 27)
        .rating-wrapper
          padding: 0 18px
          .rating-item
            position: relative
            padding: 16px 0
            border-1px(rgba(7, 17, 27, .1))
            .user
              position: absolute
              right: 0
              top: 16px
              line-height: 12px
              font-size: 0
              .name
                display: inline-block
                vertical-align: top
                font-size: 10px
                color: rgb(147, 153, 159)
              .avatar
                margin-left: 6px
                border-radius: 50%
            .time
              margin-bottom: 6px
              line-height: 12px
              font-size: 10px
              color: rgb(147, 153, 159)
            .text-wrapper
              line-height: 16px
              color: rgb(147, 153, 159)
              .icon-thumb_up, .icon-thumb_down
                display: inline-block
                margin-right: 4px
                vertical-align: top
                line-height: 16px
                font-size: 12px
              .icon-thumb_up
                color: rgb(0, 160, 220)
              .text
                display: inline-block
                vertical-align: top
                font-size: 12px
          .no-rating
            padding: 16px 0
            line-height: 12px
            font-size: 12px
            color: rgb(147, 153, 159)
</style>
