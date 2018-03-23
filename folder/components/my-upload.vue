<template>
<div>
	<input @change="addFile" :multiple="multiple" type="file" :name="name" id="my-upload"/>
</div>
</template>

<script>
export default {
	name: 'my-upload',
	props: {
		name: String,
		action: {
			type: String,
			required: true
		},
		fileList: {
			type: Array,
			default: []
		},
		data: Object,
		multiple: Boolean,
		limit: Number,
		onChange: Function,
		onBefore: Function,
		onProgress: Function,
		onSuccess: Function,
		onFailed: Function,
		onFinished: Function
	},
	methods: {
		addFile({target: {files}}){
			for(let i = 0, l = files.length; i < l; i++){
				files[i].url = URL.createObjectURL(files[i]);
				files[i].status = 'ready';
				files[i].percent = 0;
			}
			let fileList = [...this.fileList];
			if(this.multiple){
				fileList = [...fileList, ...files];
				let l = fileList.length;
				let limit = this.limit;
				if(limit && typeof limit === "number" && Math.ceil(limit) > 0 && l > limit){
					limit = Math.ceil(limit);
//					limit = limit > 10 ? 10 : limit;
					fileList = fileList.slice(l - limit);
				}
			}else{
				fileList = [files[0]];
			}
			this.onChange(fileList);
		},
		remove(index){
			let fileList = [...this.fileList];
			if(fileList.length){
				fileList.splice(index, 1);
				this.onChange(fileList);
			}
		},
		submit(){
			if(this.checkIfCanUpload()){
				if(this.onProgress && typeof XMLHttpRequest !== 'undefined')
					this.xhrSubmit();
				else
					this.fetchSubmit();
			}
		},
		fetchSubmit(){
			let keys = Object.keys(this.data), values = Object.values(this.data), action = this.action;
			const promises = this.fileList.map(each => {
				each.status = "uploading";
				let data = new FormData();
				data.append(this.name || 'file', each);
				keys.forEach((one, index) => data.append(one, values[index]));
				return fetch(action, {
				  method: 'POST',
				  headers: {
				  	"Content-Type" : "application/x-www-form-urlencoded"
				  },
				  body: data
				}).then(res => res.text()).then(res => JSON.parse(res));
			});
			Promise.all(promises).then(resArray => {
				let success = 0, failed = 0;
				resArray.forEach((res, index) => {
					if(res.code == 1){
						success++;
						this.onSuccess(index, res);
					}else if(res.code == 520){
						failed++;
						this.onFailed(index, res);
					}
				});
				return { success, failed };
			}).then(this.onFinished);
		},
		checkIfCanUpload(){
			return this.fileList.length ? (this.onBefore && this.onBefore() || !this.onBefore) : false;
		},
		xhrSubmit(){
			const _this = this;
			let options = this.fileList.map((rawFile, index) => ({
				file: rawFile,
				data: _this.data,
        filename: _this.name || "file",
        action: _this.action,
        onProgress(e){
          _this.onProgress(index, e);
        },
        onSuccess(res){
          _this.onSuccess(index, res);
        },
        onError(err){
          _this.onFailed(index, err);
        }
			}));
			let l = this.fileList.length;
			let send = async options => {
				for(let i = 0; i < l; i++){
					await _this.sendFetch(options[i]);
				}
			};
			send(options);
		},
		getBody(xhr) {
		  var text = xhr.responseText || xhr.response;
		  if (!text) {
		    return text;
		  }
	
		  try {
		    return JSON.parse(text);
		  } catch (e) {
		    return text;
		  }
		},
		sendFetch(option){
			const _this = this;
      return upload(option);
      
			function getError(action, option, xhr) {
			  var msg = void 0;
			  if (xhr.response) {
			    msg = xhr.status + ' ' + (xhr.response.error || xhr.response);
			  } else if (xhr.responseText) {
			    msg = xhr.status + ' ' + xhr.responseText;
			  } else {
			    msg = 'fail to post ' + action + ' ' + xhr.status;
			  }
		
			  var err = new Error(msg);
			  err.status = xhr.status;
			  err.method = 'post';
			  err.url = action;
			  return err;
			}
			function upload(option) {
			  if (typeof XMLHttpRequest === 'undefined') {
			    return;
			  }
				
        
			  var xhr = new XMLHttpRequest();
			  var action = option.action;
		
			  if (xhr.upload) {
			    xhr.upload.onprogress = function progress(e) {
			    	e.status = "uploading";
			      if (e.total > 0) {
			        e.percent = e.loaded / e.total * 100;
			      }
			      if(e.loaded == e.total || e.percent === 100){
			      	e.status = "finish";
			      }
			      option.onProgress(e);
			    };
			  }
		
			  var formData = new FormData();
		
			  if (option.data) {
			    Object.keys(option.data).map(function (key) {
			      formData.append(key, option.data[key]);
			    });
			  }
		
			  formData.append(option.filename, option.file);
		
			  xhr.onerror = function error(e) {
			  	e.status = "error";
			    option.onError(e);
			  };
		
			  xhr.onload = function onload() {
			    if (xhr.status < 200 || xhr.status >= 300) {
			      return option.onError(getError(action, option, xhr));
			    }
		
			    option.onSuccess(_this.getBody(xhr));
			  };
		
			  xhr.open('post', action, true);
		
			  if (option.withCredentials && 'withCredentials' in xhr) {
			    xhr.withCredentials = true;
			  }
		
			  var headers = option.headers || {};
		
			  for (var item in headers) {
			    if (headers.hasOwnProperty(item) && headers[item] !== null) {
			      xhr.setRequestHeader(item, headers[item]);
			    }
			  }
			  xhr.send(formData);
			  return xhr;
			}
		}
	}
}
</script>

<style lang="scss" scoped>
div{
	display: none;
}
</style>
