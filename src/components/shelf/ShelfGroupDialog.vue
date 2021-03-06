<template>
  <ebook-dialog :title="title" ref="dialog">
    <!-- 替换dialog中插槽的位置 -->
    <div class="dialog-list-wrapper" v-if="!ifNewGroup">
      <div class="dialog-list-item"
           :class="{'is-add': item.edit  ? item.edit === 1 : false}"
           v-for="(item, index) in categoryList"
           :key="index"
           @click="onGroupClick(item)"
           v-if="(item.edit === 2 && isInGroup) || item.edit !== 2 || !item.edit">
        <div class="dialog-list-item-text">{{item.title}}</div>
        <div class="dialog-list-icon-wrapper" v-if="isInGroup && shelfCategory.id === item.id">
          <span class="icon-check"></span>
        </div>
      </div>
    </div>
    <!-- 新增分组时展现 -->
    <div class="dialog-new-group-wrapper" v-else>
      <!-- 标题 -->
      <div class="dialog-input-title-wrapper">
        <span class="dialog-input-title">{{$t('shelf.groupName')}}</span>
      </div>
      <!-- 输入新分组名称 -->
      <div class="dialog-input-wrapper">
        <div class="dialog-input-inner-wrapper">
          <input type="text" class="dialog-input" v-model="newGroupName" ref="dialogInput">
          <div class="dialog-input-clear-wrapper" @click="clear" v-show="newGroupName && newGroupName.length > 0">
            <span class="icon-close-circle-fill"></span>
          </div>
        </div>
      </div>
    </div>
    <!-- 替换dialog中btn的默认内容 -->
    <div slot="btn" class="group-dialog-btn-wrapper">
      <div class="dialog-btn" @click="hide">{{$t('shelf.cancel')}}</div>
      <div class="dialog-btn" @click="createNewGroup" :class="{'is-empty': newGroupName && newGroupName.length === 0}"
           v-if="ifNewGroup">{{$t('shelf.confirm')}}
      </div>
    </div>
  </ebook-dialog>
</template>

<script>
  import EbookDialog from '../common/Dialog'
  import { storeShelfMixin } from '../../utils/mixin'
  import { removeAddFromShelf, appendAddToShelf, computeId } from '../../utils/store'
  import { saveBookShelf } from '../../utils/localstorage'

  export default {
    name: 'group-dialog',
    mixins: [storeShelfMixin],
    components: {
      EbookDialog
    },
    props: {
      showNewGroup: {
        type: Boolean,
        default: false
      },
      groupName: String
    },
    computed: {
      // 是否在分组书架中
      isInGroup() {
        return this.currentType === 2
      },
      // 默认的dialog列表数据
      defaultCategory() {
        return [
          {
            // 新增分组
            title: this.$t('shelf.newGroup'),
            edit: 1
          },
          {
            // 移除分组
            title: this.$t('shelf.groupOut'),
            edit: 2
          }
        ]
      },
      // 分组书架书籍数据
      category() {
        return this.shelfList.filter(item => item.type === 2)
      },
      // 将默认分组书架数据与现有的分组书架数据合并
      categoryList() {
        return [...this.defaultCategory, ...this.category]
      },
      title() {
        return !this.ifNewGroup ? this.$t('shelf.moveBook') : this.$t('shelf.newGroup')
      }
    },
    data() {
      return {
        ifNewGroup: false,
        newGroupName: ''
      }
    },
    methods: {
      show() {
        // 接收从分组书架title传来的showNewGroup和groupName重新给分组命名
        // 并将新分组设为true
        this.ifNewGroup = this.showNewGroup
        this.newGroupName = this.groupName
        this.$refs.dialog.show()
      },
      hide() {
        this.$refs.dialog.hide()
        // 上面的hide有0.2秒的动画延迟
        setTimeout(() => {
          this.ifNewGroup = false
        }, 200)
      },
      // 分组列表执行方法
      onGroupClick(item) {
         console.log(this.categoryList)
        // 当为1时创建新的分组
        if (item.edit && item.edit === 1) {
          this.ifNewGroup = true
        } else if (item.edit && item.edit === 2) {
          // 当为2时移除分组
          // console.log(this.categoryList)
          this.moveOutFromGroup(item)
        } else {
          // 移动到选择的分组
          this.moveToGroup(item)
        }
      },
      clear() {
        this.newGroupName = ''
      },
      // 移动到分组
      moveToGroup(group) {
        this.setShelfList(this.shelfList
          .filter(book => {
            // 将选择的书籍从书籍列表或分组书籍列表去掉
            if (book.itemList) {
              book.itemList = book.itemList.filter(subBook => this.shelfSelected.indexOf(subBook) < 0)
            }
            return this.shelfSelected.indexOf(book) < 0
          }))
          .then(() => {
            // 将当前分组书籍列表与被选择的书籍列表进行合并
            if (group && group.itemList) {
              group.itemList = [...group.itemList, ...this.shelfSelected]
            }
            // 刷新id
            group.itemList.forEach((item, index) => {
              item.id = index + 1
            })
            this.simpleToast(this.$t('shelf.moveBookInSuccess').replace('$1', group.title))
            this.onComplete()
          })
      },
      // 移除分组
      moveOutFromGroup(item) {
        // this.moveOutOfGroup(this.onComplete)
        // console.log(item)
        // 遍历筛选出没有被选择的书籍
        this.shelfList.map(book => {
          if(book.type === 2 && book.itemList) {
            book.itemList = book.itemList.filter(subBook => !subBook.selected)
          }
          return book
        }).then(() => {
          // 筛选完后移除主书架中的添加图案
          let list = removeAddFromShelf(this.shelfList)
          // 并将选择去除的书籍与主书架中的书籍进行合并
          list = [].concat(list,...this.shelfSelected)
          // 重新添加添加图案
          list = appendAddToShelf(list)
          // 将书架中的书籍重新排序
          list = computeId(list)
          // 更新vuex中书架数据
          this.setShelfList(list).then(() => {
            this.simpleToast(this.$t('shelf.moveBookoutSuccess'))
            this.onComplete()
          })

        })

      },
      // 新建分组
      createNewGroup() {
        // 不存在新分组名字，或者为空
        if (!this.newGroupName || this.newGroupName.length === 0) {
          return
        }
        // 当存在新分组时修改分组的title
        if (this.showNewGroup) {
          this.shelfCategory.title = this.newGroupName
          this.onComplete()
        } else {
          // 当不存在时创建新分组
          const group = {
            // id为书架书籍列表中的倒数第二个再加一
            id: this.shelfList[this.shelfList.length - 2].id + 1,
            itemList: [],
            selected: false,
            title: this.newGroupName,
            type: 2
          }
          // 移除加号
          let list = removeAddFromShelf(this.shelfList)
          // 将新分组添加到书架列表中
          list.push(group)
          // 重新添加加号
          list = appendAddToShelf(list)
          // 更新主书架数据
          this.setShelfList(list).then(() => {
            // 将选择到的书籍直接加入分组中
            this.moveToGroup(group)
          })
        }
      },
      onComplete() {
        saveBookShelf(this.shelfList)
        this.hide()
        this.setIsEditMode(false)
      }
    }
  }
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
  @import "../../assets/styles/global";

  .dialog-list-wrapper {
    width: 100%;
    padding: 0 px2rem(20);
    box-sizing: border-box;
    font-size: px2rem(14);
    @include scroll;
    .dialog-list-item {
      display: flex;
      padding: px2rem(15) 0;
      box-sizing: border-box;
      color: #666;
      &.is-add {
        color: $color-blue;
        &:active {
          color: $color-blue-transparent;
        }
      }
      &:active {
        color: rgba(102, 102, 102, .5)
      }
      .dialog-list-item-text {
        flex: 1;
      }
      .dialog-list-icon-wrapper {
        flex: 0 0 px2rem(30);
        color: $color-blue;
        @include right;
      }
    }
  }

  .dialog-new-group-wrapper {
    width: 100%;
    padding: 0 px2rem(20);
    box-sizing: border-box;
    .dialog-input-title-wrapper {
      font-size: px2rem(10);
      margin-top: px2rem(20);
    }
    .dialog-input-wrapper {
      width: 100%;
      padding: 0 0 px2rem(30) 0;
      box-sizing: border-box;
      .dialog-input-inner-wrapper {
        display: flex;
        width: 100%;
        padding: px2rem(10) 0;
        box-sizing: border-box;
        border-bottom: px2rem(1) solid #eee;
        font-size: px2rem(14);
        color: #666;
        .dialog-input {
          flex: 1;
          border: none;
          &:focus {
            outline: none;
          }
        }
        .dialog-input-clear-wrapper {
          flex: 0 0 px2rem(30);
          color: #ccc;
          @include center;
          &:active {
            color: #999;
          }
        }
      }
    }
  }

  .group-dialog-btn-wrapper {
    width: 100%;
    @include center;
  }

  .dialog-btn {
    flex: 1;
    &.is-empty {
      color: rgba(255, 255, 255, .5);
    }
    &:active {
      color: rgba(255, 255, 255, .5)
    }
  }
</style>
