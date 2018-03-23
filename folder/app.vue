<template>
  <div class="content">
		<label for="my-upload">
			<span>上传</span>
		</label>
    <my-upload
	    ref="myUpload"
	    :file-list="fileList"
	    action="/user/uploadUserPicture"
	    :data="param"
	    :on-change="onChange"
	    :on-progress="uploadProgress"
	    :on-success="uploadFinish"
	    :on-failed="uploadFailed"
	    :on-finished="onFinished">
    </my-upload>
    <button @click="upload" class="btn btn-xs btn-primary">Upload</button>
    <div class="data-show">
    	<div v-for="({url, percent}, index) in fileList"><img :src="url"/>
    	<span class="progress-span">{{percent}}%</span>
    	<i @click="removeFile(index)">X</i>
    	</div>
    </div>
  </div>
</template>

<script>
import myUpload from './components/my-upload'
export default {
  name: 'test',
  data(){
  	return {
  		fileList: [],
  		param: {identity: '410184199203061254', isUpdate: '0'},
  	}
  },
  methods: {
  	onChange(fileList){
  		this.fileList = [...fileList];
  		console.log(fileList);
  	},
  	uploadFinish(index, response){
  		console.log(index, response);
  	},
  	upload(){
  		this.$refs.myUpload.submit();
  	},
  	removeFile(index){
  		this.$refs.myUpload.remove(index);
  	},
  	uploadProgress(index, progress){
  		const { percent, status } = progress;
  		const fileList = [...this.fileList];
  		fileList[index].percent = percent.toFixed(2);
  		this.fileList = [...fileList];
  		console.log(index, percent, status);
  	},
  	uploadFailed(index, err){
  		console.log(index, err);
  	},
  	onFinished(result){//result: { success, failed }
  		console.log(result);
  	}
  },
  components: {
  	myUpload
  }
}
</script>
<style lang="scss" scoped>
.content{
	height: 100%;
	overflow-y: scroll;
	overflow-x: hidden;
	background: linear-gradient(to bottom, #f4f4f4, #fff);
	background: -moz-linear-gradient(to bottom, #f4f4f4, #fff);
	background: -o-linear-gradient(to bottom, #f4f4f4, #fff);
}
.data-show{
	overflow: auto;
	display: flex;
	flex-flow: row wrap;
	justify-content: center;
	div{
		width: 4rem;
		height: 4rem;
		margin: 0.5rem;
		position: relative;
		img{
			width: 100%;
			height: 100%;
		}
		span{
			position: absolute;
			color: #fff;
			top: 50%;
			left: 50%;
			transform: translate(-50%,-50%);
		}
		i{
			position: absolute;
			top: 0;
			right: 0;
			font-style: normal;
		}
	}
}
</style>
