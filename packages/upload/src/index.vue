<script>
import UploadList from './upload-list';
import Upload from './upload';
import ElProgress from 'packages/progress/index.js';

function noop() {
}

export default {
  name: 'el-upload',

  // extends: typeof FormData !== 'undefined' ? ajaxUpload : iframeUpload,
  // extends: iframeUpload,

  components: {
    ElProgress,
    UploadList,
    Upload
  },

  props: {
    action: {
      type: String,
      required: true
    },
    headers: {
      type: Object,
      default() {
        return {
          // 'Access-Control-Request-Methods': 'GET, PUT, POST, DELETE, OPTIONS',
          // 'Access-Control-Request-Headers': 'Content-Type, Content-Range, Content-Disposition, Content-Description'
        };
      }
    },
    multiple: {
      type: Boolean,
      default: false
    },
    name: {
      type: String,
      default: 'file'
    },
    withCredentials: {
      type: Boolean,
      default: false
    },
    thumbnailMode: Boolean,
    showUploadList: {
      type: Boolean,
      default: true
    },
    accept: String,
    type: {
      type: String,
      default: 'select'
    },
    beforeUpload: Function,
    onRemove: {
      type: Function,
      default: noop
    },
    onChange: {
      type: Function,
      default: noop
    },
    onPreview: {
      type: Function,
      default: noop
    }
  },

  data() {
    return {
      uploadedFiles: [],
      dragOver: false,
      draging: false,
      tempIndex: 1
    };
  },

  methods: {
    onStart(file) {
      file.uid = Date.now() + this.tempIndex++;
      let _file = {
        status: 'uploading',
        name: file.name,
        size: file.size,
        percentage: 0,
        uid: file.uid,
        showProgress: true
      };

      if (this.thumbnailMode) {
        try {
          _file.url = URL.createObjectURL(file);
        } catch (err) {
          console.log(err);
          return;
        }
      }

      this.uploadedFiles.push(_file);
    },
    onProgress(ev, file) {
      var _file = this.getFile(file);
      _file.percentage = ev.percent;
    },
    onSuccess(res, file) {
      var _file = this.getFile(file);

      _file.status = 'finished';
      _file.response = res;

      setTimeout(() => {
        _file.showProgress = false;
      }, 1000);
    },
    onError(err, file) {
      var _file = this.getFile(file);
      var fileList = this.uploadedFiles;

      _file.status = 'fail';

      fileList.splice(fileList.indexOf(_file), 1);
      this.$emit('error', _file, fileList, err);
    },
    handleRemove(file) {
      var fileList = this.uploadedFiles;
      fileList.splice(fileList.indexOf(file), 1);
      this.onRemove(file, fileList);
    },
    getFile(file) {
      var fileList = this.uploadedFiles;
      var target;
      fileList.every(item => {
        target = file.uid === item.uid ? item : null;
        return !target;
      });
      return target;
    },
    handlePreview(file) {
      if (file.status === 'finished') {
        this.onPreview(file);
      }
    }
  },

  render(h) {
    var uploadList;
    if (this.showUploadList && !this.thumbnailMode) {
      uploadList = (
        <UploadList
          files={this.uploadedFiles}
          on-remove={this.handleRemove}
          on-preview={this.handlePreview}>
        </UploadList>
      );
    }

    var props = {
      props: {
        action: this.action,
        multiple: this.multiple,
        'before-upload': this.beforeUpload,
        'with-credentials': this.withCredentials,
        name: this.name,
        accept: this.thumbnailMode ? 'image/*' : this.accept,
        'on-start': this.onStart,
        'on-progress': this.onProgress,
        'on-success': this.onSuccess,
        'on-error': this.onError,
        'on-preview': this.handlePreview,
        'on-remove': this.handleRemove
      }
    };

    if (this.type === 'select') {
      return (
        <div class="el-upload">
          {uploadList}
          <upload {...props}>
            {this.$slots.default}
          </upload>
          {this.$slots.tip}
        </div>
      );
    }

    if (this.type === 'drag') {
      props.props.type = 'drag';

      return (
        <div class="el-upload">
          <upload {...props}>
            {this.$slots.default}
          </upload>
          {this.$slots.tip}
          {uploadList}
        </div>
      );
    }
  }
};
</script>