<template>
	<div id="app">
		<router-view />
		<el-dialog
			title="待办提醒"
			:visible.sync="dialogVisible"
			width="30%"
			:show-close='false'
			>
			<p>{{nowTodo.content}}</p>
			<p>{{nowTodo.timestamp}}</p>
			<span slot="footer" class="dialog-footer">
				<el-button  type="primary" @click="noRemind">不再提醒</el-button>
				<!-- <el-button type="primary" @click="dialogVisible = false">确 定</el-button> -->
			</span>
		</el-dialog>
		
	</div>
	
</template>

<script>
	const {ipcRenderer} = require('electron')
	ipcRenderer.on('close-win', (event, arg) => {
		localStorage.clear()
		// alert(arg) // prints "pong"
	})
	import { getToken} from '@/utils/auth'
	import { mapGetters } from 'vuex'
	export default {
		name: 'App',
		data(){
			return{
				dialogVisible: false,
				timer:null,
				nowTodo:'',
			}
		},
		mounted(){
			this.getList()
			console.log(this.todolist)
		},
		computed: {
			...mapGetters([
				'todolist'
			])
		},
		watch:{
			'todolist':function(n, o){
				console.log(n)
				window.clearInterval(this.timer)
				// console.log('n------------')
				for (let i = 0; i < n.length; i++) {
					if (n[i].remind == '0') {
						console.log(n[i])
						this.nowTodo = n[i];
						var now = new Date().getTime()
						var todo = new Date(n[i].timestamp).getTime()
						this.timer = window.setInterval(() => {
							now = new Date().getTime()
							// console.log(new Date())
							if(now >= todo){
								this.dialogVisible =true;
							}
						}, 1000);
						break;
					}else{
						window.clearInterval(this.timer)
					}
				}
			}
		},
		methods:{
			getList(){
				var timer=null;
				this.$http({
					url: '/login/todo/getTodoList',
					method: 'post',
					data:{
						uid:getToken(),
						page:1,
						pageSize:10
					}
				}).then((res) => {
					this.$store.dispatch('setList',res.list)
				})
			},
			noRemind(){
				this.dialogVisible = false;
				// this.timer = null;
				window.clearInterval(this.timer)
				this.$http({
					url: '/login/todo/updateRemind',
					method: 'post',
					data:{
						uid:getToken(),
						id:this.nowTodo.id
					}
				}).then((res) => {
					this.getList()
				})
			},
		}
	}
</script>
<style lang="scss">
  @import './styles/index.scss'; // 全局自定义的css样式
</style>