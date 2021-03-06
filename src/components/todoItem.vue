<template>
	<div class="card todo-item">
		<div class="content todo-item__content" v-show="!todo.isEditing">
			<span class="item-action" v-if="isThreadCreator()">
				<i class="right floated basic red trash icon" v-on:click="removeItem()"></i>
				<i class="right floated basic yellow edit icon" v-on:click="toggleIsEditing()"></i>	
			</span>
			
			<div class="header">
				<router-link :to="`/detail/${todo.id}`">{{ todo.title }}</router-link>
			</div>
			<div class="meta">
				<i class="red lock icon" v-show="todo.isDone"></i>
				<i class="green unlock icon" v-show="!todo.isDone"></i>
				{{ todo.isDone ? 'Closed' : 'Opened' }}
			</div>
			<div class="description">
				{{ `${todo.desc.slice(0, 200)}...` }}
			</div>
			<div class="info">
				Created by <span class="info-meta__user">{{todo.createdBy}}</span> at <span class="info-meta__date">{{ this.calculateDiffTime(todo.createdAt) }}</span>
			</div>
		</div>
		<!-- Form Todo Item -->
		<div class="content" v-show="todo.isEditing">
			<div class="ui form">
				<div class="field">
					<label>Title</label>
					<input type="text" name="todo-title" v-model="todo.title">
				</div>
				<div class="field">
					<label>Description</label>
					<textarea name="todo-desc" v-model="todo.desc"></textarea>
				</div>
				<div class="ui two button attached buttons">
					<button class="ui basic grey button" v-on:click="editItem()">
						Finish Editing
					</button>
				</div>
			</div>
		</div>

		<div class="extra content todo-item__actions" v-show="!todo.isEditing">
			<div class="ui labeled button" tabindex="0" v-show="!todo.isDone">
				<div class="ui vertical animated red basic button" v-on:click="toggleStatus()">
					<div class="visible content">
						Mark as Closed
					</div>
					<div class="hidden content">
						<i class="lock icon"></i>
					</div>
				</div>
				<a href="#" class="ui basic red left pointing label">
					<router-link :to="`/detail/${todo.id}`" class="ui red">
						<i class="comments icon"></i>
						{{ comments }}
					</router-link>
				</a>	
			</div>
			
			<div class="ui labeled button" tabindex="0" v-show="todo.isDone">
				<div class="ui vertical animated green button" v-on:click="toggleStatus()">
					<div class="visible content">
						Mark as Opened
					</div>
					<div class="hidden content">
						<i class="unlock icon"></i>
					</div>
				</div>
				<a href="#" class="ui basic green left pointing label">
					<router-link :to="`/detail/${todo.id}`" class="ui green">
						<i class="comments icon"></i>
						{{ comments }}
					</router-link>
				</a>
			</div>
		</div>
	</div>
</template>

<script type="text/javascript">
	//utils
	import { getCurrentUsername, removeTodo, setTodoStatus, updateTodo } from '../utils/api/todo';
	import { calculateDiffTime, objectListToArray } from '../utils/helpers/stringManipulation';
	import firebase from 'firebase';

	export default {
		name: 'TodoItem',
		props: ['todo'],
		created: function(){
			let dbComments = firebase.database().ref('comments')

			dbComments.child(this.todo.id).on('value', (res) => {
				if(!res.val()){
					this.comments = 0;
				} else {
					this.comments = objectListToArray(res.val()).length
				}
			});
		},
		data: function(){
			return {
				comments: 0
			}
		},
		methods: {
			calculateDiffTime(time){
				return calculateDiffTime(time)
			},
			editItem(){
				let item = this.todo,
					{ key } = item;

				delete item.key;
				delete item.id;

				updateTodo(key, Object.assign({}, item, {isEditing: false}));
				this.toggleIsEditing();
			},
			isThreadCreator(){
				return this.todo.createdBy === getCurrentUsername()
			},
			toggleIsEditing(){
				this.todo.isEditing = !this.todo.isEditing;
			},
			toggleStatus(){
				if(this.isThreadCreator()){
					this.todo.isDone = !this.todo.isDone;
					setTodoStatus(this.todo.id, this.todo.isDone)
				}
			},
			removeItem(){
				removeTodo(this.todo.id)
			}
		}
	}
</script>

<style lang="scss">
	.todo-item {
		&__content {

			.item-action {
				cursor: pointer;
				font-size: 1.5em;
			}

			.header {
				text-decoration: underline;
				font-size: 1.3em !important;
			}

			.description {
				text-align: justify;
				text-indent: 1.8em;
			}

			.info {
				margin-top: 9px;
				padding-top: 6px;
				font-size: 0.8em;
				border-top: 1px dashed #f0f0f0;

				&-meta {
					&__user {
						font-weight: 500;
					}

					&__date {
						font-style: italic;
					}
				}
			}
		}

		&__actions {
			text-align: center;
		}
	}
</style>