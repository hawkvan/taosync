<template>
	<div class="login">
		<div class="loginArea">
			<div class="header">
				<div class="logo">TaoSync</div>
				<div class="title">自动同步工具</div>
			</div>
			
			<el-form :model="loginForm" :rules="rules" ref="loginForm" label-width="0">
				<el-form-item prop="userName">
					<el-input class="custom-input" placeholder="用户名" prefix-icon="el-icon-user"
						v-model="loginForm.userName"></el-input>
				</el-form-item>
				<el-form-item prop="passwd">
					<el-input class="custom-input" placeholder="密码" prefix-icon="el-icon-lock" v-model="loginForm.passwd" show-password
						@keyup.enter.native="login"></el-input>
				</el-form-item>
			</el-form>
			
			<div class="action-bar">
				<div class="foget" @click="fogetPwd">忘记密码？</div>
			</div>

			<el-button class="login-button" type="primary" :loading="loading" @click="login">登 录</el-button>
		</div>

		<el-dialog :close-on-click-modal="false" :visible.sync="showPwd" :append-to-body="true" title="重置密码"
			width="90%" class="custom-dialog" :before-close="closePwd">
			<el-form :model="pwdForm" :rules="pwdRules" ref="resetForm" label-position="top">
				<el-form-item prop="userName" label="用户名">
					<el-input placeholder="请输入用户名" v-model="pwdForm.userName"></el-input>
				</el-form-item>
				<el-form-item prop="key" label="加密秘钥">
					<el-input type="textarea" :rows="2" placeholder="复制 data/secret.key 内容" v-model="pwdForm.key"></el-input>
				</el-form-item>
				<el-form-item prop="passwd" label="新密码">
					<el-input placeholder="请输入新密码" v-model="pwdForm.passwd" show-password></el-input>
				</el-form-item>
				<el-form-item prop="passwd2" label="确认密码">
					<el-input placeholder="请确认新密码" v-model="pwdForm.passwd2" show-password></el-input>
				</el-form-item>
			</el-form>
			<span slot="footer" class="dialog-footer">
				<el-button @click="closePwd" size="small">取 消</el-button>
				<el-button type="primary" @click="fogetSubmit" :loading="loading" size="small">确 定</el-button>
			</span>
		</el-dialog>
	</div>
</template>

<script>
/* ... script 逻辑保持不变，只需直接复用你原有的逻辑即可 ... */
import Cookies from 'js-cookie';
import { login, resetPwd } from "@/api/user";
export default {
    name: 'Login',
    data() {
        var validatePass2 = (rule, value, callback) => {
            if (value == '' || value == null) {
                callback(new Error('请再次输入新密码'));
            } else if (value !== this.pwdForm.passwd) {
                callback(new Error('两次输入密码不一致!'));
            } else {
                callback();
            }
        };
        return {
            loginForm: { userName: null, passwd: null },
            loading: false,
            rules: {
                userName: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
                passwd: [{ required: true, message: '请输入密码', trigger: 'blur' }]
            },
            pwdRules: {
                userName: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
                key: [{ required: true, message: '请输入key', trigger: 'blur' }],
                passwd: [{ required: true, message: '请输入新密码', trigger: 'blur' }],
                passwd2: [{ validator: validatePass2, trigger: 'blur' }]
            },
            showPwd: false,
            pwdForm: { userName: null, key: null, passwd: null, passwd2: null }
        };
    },
    methods: {
        login() {
            this.$refs.loginForm.validate((valid) => {
                if (valid) {
                    Cookies.remove(this.vuex_cookieName);
                    this.loading = true;
                    login(this.loginForm).then(res => {
                        this.$setVuex('vuex_userInfo', res.data);
                        this.$router.replace('/home');
                        this.loading = false;
                    }).catch(() => { this.loading = false; })
                }
            });
        },
        fogetPwd() { this.showPwd = true; },
        closePwd() { this.showPwd = false; this.pwdForm = { userName: null, key: null, passwd: null, passwd2: null } },
        fogetSubmit() {
            this.$refs.resetForm.validate((valid) => {
                if (valid) {
                    this.loading = true;
                    resetPwd(this.pwdForm).then(res => {
                        this.closePwd();
                        this.$message({ message: '密码重置成功', type: 'success' });
                        this.loading = false;
                    }).catch(() => { this.loading = false; })
                }
            });
        }
    }
}
</script>

<style lang="scss" scoped>
.login {
	display: flex;
	align-items: center;
	justify-content: center;
	position: fixed;
	top: 0; bottom: 0; left: 0; right: 0;
	background: #f5f5f7; // 浅灰背景，干净清爽

	.loginArea {
		background: #ffffff;
		width: 100%;
		max-width: 400px; // 适应移动端
		margin: 20px;
		padding: 40px 30px;
		border-radius: 20px;
		box-shadow: 0 10px 40px rgba(0, 0, 0, 0.04);

		.header {
			text-align: center;
			margin-bottom: 40px;

			.logo {
				font-size: 28px;
				font-weight: 600;
				color: #1d1d1f;
				letter-spacing: -0.5px;
			}
			.title {
				font-size: 14px;
				color: #86868b;
				margin-top: 8px;
			}
		}

		:deep(.custom-input) {
			.el-input__inner {
				background-color: #f5f5f7;
				border: none;
				height: 50px;
				border-radius: 12px;
				font-size: 15px;
				color: #1d1d1f;
				transition: all 0.2s;
				
				&:focus {
					background-color: #ffffff;
					box-shadow: 0 0 0 2px #007aff;
				}
			}
			.el-input__prefix {
				left: 10px;
				line-height: 50px;
			}
		}

		.action-bar {
			display: flex;
			justify-content: flex-end;
			margin-top: -10px;
			margin-bottom: 25px;

			.foget {
				font-size: 13px;
				color: #007aff;
				cursor: pointer;
				&:hover { text-decoration: underline; }
			}
		}

		.login-button {
			width: 100%;
			height: 50px;
			border-radius: 12px;
			font-size: 16px;
			font-weight: 600;
			background: #007aff;
			border: none;
			box-shadow: 0 4px 12px rgba(0, 122, 255, 0.3);
		}
	}
}

// 弹窗适配移动端
:deep(.custom-dialog) {
	.el-dialog {
		border-radius: 16px;
		max-width: 500px;
	}
	.el-dialog__header { font-weight: 600; }
}
</style>
