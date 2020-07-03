<template>
	<view class="todo">
		<view class="header" v-if="list.length !== 0">
			<view class="header-left">
				<text class="active-text">{{currentpage}}</text>
				<text>{{listData.length}} 条</text>
			</view>
			<view class="header-right">
				<view class="header-item" :class="{'active-tab':activeIndex === 0}" @click="tab(0)">全部</view>
				<view class="header-item" :class="{'active-tab':activeIndex === 1}" @click="tab(1)">待办</view>
				<view class="header-item" :class="{'active-tab':activeIndex === 2}" @click="tab(2)">已完成</view>
			</view>
		</view>
		<view class="default" v-if="list.length === 0">
			<view class="image-default">
				<image src="../../static/default.png" mode="aspectFit"></image>
			</view>
			<view class="default-info">
				<view class="default-info-text">您还没有创建任何待办事项</view>
				<view class="default-info-text">点击下方 + 号创建一个吧</view>
			</view>
		</view>

		<view class="content">
			<view class="todo-list" :class="{'todo-finish':item.checked}" v-for="item in listData" :key=item.id>
				<view class="todo-list-checkbox" @click="finish(item.id)">
					<view class="checkbox"></view>
				</view>
				<view class="todo-list-content" @click="finish(item.id)">
					{{item.content}}
				</view>
				<view class="deleteTodo">
					<text class="iconfont icon-lajitong" @click="deleteTodo(item.id)"></text>
				</view>
			</view>

			<view class="create-todo" @click="openCreate">
				<text class="iconfont icon-jiahao" :class="{'create-todo-active':add_animation}"></text>
			</view>
			<view class="create-content" :class="{'create-show':add_animation}" v-show="active">
				<view class="create-content-box">
					<view class="create-input">
						<input type="text" v-model="value" placeholder="请输入待办事项" />
					</view>
					<view class="create-button" @click="createToDo">
						创建
					</view>
				</view>
			</view>

		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				active: false,
				value: "",
				activeIndex: 0,
				currentpage: '全部',
				add_animation: false,
				list2: [],
				list: []
			}
		},
		onLoad() {

		},
		mounted() {
			if (uni.getStorageSync('list') == '') {
				uni.setStorageSync('list', JSON.stringify([]))
				this.list = JSON.parse(uni.getStorageSync('list'))
				console.log(this.list)
			} else {
				
				this.list = JSON.parse(uni.getStorageSync('list'))
			}
		},
		computed: {
			listData() {
				let newList = []
				switch (this.activeIndex) {
					case 0:
						return this.list
						break;
					case 1:
						this.list.forEach(item => {
							if (!item.checked) {
								newList.push(item)
							}
						})
						return newList
						break;
					case 2:
						this.list.forEach(item => {
							if (item.checked) {
								newList.push(item)
							}
						})
						return newList
						break;
				}
			}
		},
		methods: {
			openCreate() {
				this.active === true ? this.closeAnimation() : this.openAnimation();
			},
			openAnimation() {
				this.active = !this.active,
					setTimeout(() => {
						this.add_animation = !this.add_animation
					}, 50)
			},
			closeAnimation() {
				this.add_animation = !this.add_animation
				setTimeout(() => {
					this.active = !this.active
				}, 350)
			},
			createToDo() {
				if (this.value !== "") {
					let newTodo = {
						id: this.list.length,
						content: this.value,
						checked: false
					}
					this.list.unshift(newTodo)
					uni.setStorageSync('list', JSON.stringify(this.list))
				} else {
					uni.showToast({
						title: '请输入内容',
						icon: 'none',
						duration: 2000
					})
				}
				this.value = ""
				this.closeAnimation()
			},
			finish(id) {
				let index = this.list.findIndex(item => item.id == id)
				let finishItem = this.list[index]
				this.list[index].checked = !this.list[index].checked
				if (finishItem.checked == true) {
					this.list.splice(index, 1)
					this.list.push(finishItem)
					uni.setStorageSync('list', JSON.stringify(this.list))
				}else{
					this.list.splice(index, 1)
					this.list.unshift(finishItem)
					uni.setStorageSync('list', JSON.stringify(this.list))
				}
			},
			tab(index) {
				this.activeIndex = index
				switch (index) {
					case 0:
						this.currentpage = '全部'
						break;
					case 1:
						this.currentpage = '待办'
						break;
					case 2:
						this.currentpage = '已完成'
						break;
				}
			},
			deleteTodo(id) {
				let _this = this
				uni.showModal({
					title: '确定要删除吗',
					success: function(res) {
						if (res.confirm) {
							let deleteItem = _this.list.findIndex(item => item.id == id)
							_this.list.splice(deleteItem, 1)
							uni.setStorageSync('list', JSON.stringify(_this.list))
						} else {
							return
						}
					}
				})

			}
		}
	}
</script>

<style>
	@import '../../common/icon.css';

	.header {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 45px;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 10px;
		font-size: 12px;
		color: 333;
		background: #fff;
		box-shadow: -1px 1px 5px rgba(0, 0, 0, .1);
		box-sizing: border-box;
		z-index: 100;
	}

	/* #ifdef H5*/
	.header {
		top: 44px;
	}

	/* #endif */

	.header-right {
		display: flex;
	}

	.header-right view {
		padding: 0 5px;
	}

	.active-text {
		font-size: 14px;
		color: #279abf;
		padding-right: 10px;
	}

	.active-tab {
		color: #279abf;
	}

	.content {
		position: relative;
		padding-top: 50px;
		padding-bottom: 100px;
	}

	.todo-list {
		position: relative;
		display: flex;
		align-items: center;
		padding: 15px;
		margin: 15px;
		color: #0c3854;
		font-size: 14px;
		border-radius: 10px;
		background: #cfebfd;
		box-shadow: -1px 1px 5px rgba(0, 0, 0, .1), -1px 2px 1px 0 rgb(255, 255, 255) inset;
		overflow: hidden;
	}

	.todo-list::after {
		content: '';
		position: absolute;
		top: 0;
		bottom: 0;
		left: 0;
		width: 5px;
		background: #91d1e8;
	}

	.todo-list-checkbox {
		padding-right: 15px;
	}

	.checkbox {
		width: 20px;
		height: 20px;
		border-radius: 50%;
		background: #fff;
		box-shadow: 0 0 1px 1px rgba(0, 0, 0, .1);
	}

	.todo-list-content {
		width: 100%
	}

	.todo-finish .checkbox {
		background: #eee;
		position: relative;
	}

	.todo-finish .checkbox::after {
		content: '';
		width: 10px;
		height: 10px;
		border-radius: 50%;
		position: absolute;
		top: 0;
		left: 0;
		bottom: 0;
		right: 0;
		margin: auto;
		background: #cfebfd;
		box-shadow: 0 0 2px 0 rgba(0, 0, 0, .2) inset;
	}

	.todo-finish .todo-list-content {
		color: #999;
	}

	/* 同级关系不需要空格 */
	.todo-finish.todo-list::before {
		content: '';
		position: absolute;
		top: 0;
		bottom: 0;
		left: 35px;
		right: 40px;
		height: 2px;
		margin: auto 0;
		background: #bdcdd8;
	}

	.todo-finish.todo-list::after {
		background: #bdcdd8;
	}

	.create-todo {
		display: flex;
		align-items: center;
		justify-content: center;
		position: fixed;
		bottom: 20px;
		left: 0;
		right: 0;
		margin: 0 auto;
		width: 50px;
		height: 50px;
		border-radius: 50px;
		background: #deeff5;
		box-shadow: -1px 1px 5px 2px rgba(0, 0, 0, .1), -1px 1px 1px 0 rgb(255, 255, 255)inset;
	}

	.icon-jiahao {
		font-size: 30px;
		color: #add8e6;
	}

	.create-content {
		position: fixed;
		bottom: 95px;
		left: 20px;
		right: 20px;
		opacity: 0;
		transition: all .3s;
		transform: scale(0) translateY(300%);
	}

	.create-show {
		opacity: 1;
		transform: scale(1) translateY(0);
	}

	.create-content-box {
		display: flex;
		align-items: center;
		padding-left: 15px;
		border-radius: 50px;
		background: #deeff5;
		box-shadow: -1px 1px 5px 2px rgba(0, 0, 0, .1), -1px 1px 1px 0 rgb(255, 255, 255)inset;
		z-index: 2;
	}

	.create-input {
		width: 100%;
		padding-right: 15px;
		color: #add8e6;
	}

	.create-button {
		flex-shrink: 0;
		width: 80px;
		height: 50px;
		border-radius: 50px;
		font-size: 16px;
		color: #88d4ec;
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: -2px 0 2px 1px rgba(0, 0, 0, .1);
	}

	.create-content::after {
		content: '';
		width: 20px;
		height: 20px;
		background: #deeff5;
		position: absolute;
		left: 0;
		right: 0;
		bottom: -10px;
		margin: 0 auto;
		transform: rotate(45deg);
		box-shadow: -1px 1px 5px 2px rgba(0, 0, 0, .1);
		z-index: -1;
	}

	.create-content-box::after {
		content: '';
		width: 20px;
		height: 20px;
		background: #deeff5;
		position: absolute;
		left: 0;
		right: 0;
		bottom: -10px;
		margin: 0 auto;
		transform: rotate(45deg);
	}

	.default {
		padding-top: 100px;
	}

	.image-default {
		display: flex;
		justify-content: center;
	}

	.image-default image {
		width: 100%;
	}

	.default-info {
		font-size: 16px;
		color: #ccc;
		text-align: center;
	}

	.icon-jiahao {
		transition: all .3s;
	}

	.create-todo-active {
		transform: rotate(135deg);
	}

	.deleteTodo {
		text-align: right;
	}

	.icon-lajitong {
		font-size: 18px;
		color: #348fca;
	}
</style>
