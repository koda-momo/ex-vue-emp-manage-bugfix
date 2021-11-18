<template>
  <div class="container">
    <div class="row register-page">
      <div class="error">{{ errorMessage }}</div>
      <div class="error" v-show="nameErrorMessage">＊名前を入力して下さい</div>
      <div class="error" v-show="mailErrorMessage">
        ＊メールアドレスを入力して下さい
      </div>
      <div class="error" v-show="passErrorMessage">
        ＊パスワードを入力して下さい
      </div>
      <div class="error" v-show="verificationPassErrorMessage">
        ＊パスワードが一致しません
      </div>
      <div class="error" v-show="postalCodeErrorMessage">
        ＊郵便番号を入力して下さい
      </div>
      <div class="error" v-show="addressErrorMessage">
        ＊住所を入力して下さい
      </div>

      <form class="col s12" id="reg-form">
        <!-- 名前-->
        <div class="row">
          <div class="input-field col s6">
            <input
              id="last_name"
              type="text"
              class="validate"
              v-model="lastName"
              required
            />
            <label for="last_name">姓</label>
          </div>
          <div class="input-field col s6">
            <input
              id="first_name"
              type="text"
              class="validate"
              v-model="firstName"
              required
            />
            <label for="first_name">名</label>
          </div>
        </div>
        <!-- メールアドレス-->
        <div class="row">
          <div class="input-field col s12">
            <input
              id="email"
              type="email"
              class="validate"
              v-model="mailAddress"
              required
            />
            <label for="email">メールアドレス</label>
          </div>
        </div>
        <!-- パスワード -->
        <div class="row">
          <div class="input-field col s12">
            <input
              id="password"
              type="password"
              class="validate"
              minlength="8"
              v-model="password"
              required
            />
            <label for="password">パスワード</label>
          </div>
        </div>
        <!-- 確認用パスワード -->
        <div class="row">
          <div class="input-field col s12">
            <input
              id="verificationPassword"
              type="password"
              class="validate"
              minlength="8"
              v-model="verificationPassword"
              required
            />
            <label
              for="
VerificationPassword"
              >パスワード(確認用)</label
            >
          </div>
        </div>
        <!-- 郵便番号 -->
        <div class="postCode">
          <span class="row">
            <span class="input-field col s12">
              <input
                id="postalCode"
                type="text"
                class="postalCode"
                v-model="postalCode"
                required
                minlength="7"
                maxlength="7"
              />
              <label
                for="
postalCode"
                >郵便番号(ハイフンなし)</label
              >
            </span>
            <button
              type="button"
              class="postbtn btn-large btn-register waves-effect waves-light"
            >
              住所検索
            </button>
          </span>
        </div>
        <!-- 住所 -->
        <div class="row">
          <div class="input-field col s12">
            <input
              id="address"
              type="text"
              class="address"
              v-model="address"
              required
            />
            <label
              for="
address"
              >住所</label
            >
          </div>
        </div>
        <!-- ボタン -->
        <div class="row">
          <div class="input-field col s6">
            <button
              class="btn btn-large btn-register waves-effect waves-light"
              type="button"
              v-on:click="registerAdmin"
            >
              登録
              <i class="material-icons right">done</i>
            </button>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import config from "@/const/const";
import axios from "axios";

/**
 * 管理者登録をする画面.
 */
@Component
export default class RegisterAdmin extends Vue {
  // 姓
  private lastName = "";
  // 名
  private firstName = "";
  // メールアドレス
  private mailAddress = "";
  // パスワード
  private password = "";
  //確認用パスワード
  private verificationPassword = "";
  // 郵便番号
  private postalCode = "";
  // 住所
  private address = "";
  //エラーメッセージ
  private errorMessage = "";
  //名前エラー
  private nameErrorMessage = false;
  //メールアドレスエラー
  private mailErrorMessage = false;
  //パスワードエラー
  private passErrorMessage = false;
  //確認用パスワードエラー
  private verificationPassErrorMessage = false;
  //住所エラー
  private addressErrorMessage = false;
  //郵便番号エラー
  private postalCodeErrorMessage = false;

  /**
   * 管理者情報を登録する.
   *
   * @remarks
   * 本メソッドは非同期でWebAPIを呼び出し管理者登録をするためasync/await axiosを利用しています。これらを利用する場合は明示的に戻り値にPromiseオブジェクト型を指定する必要があります。
   * @returns Promiseオブジェクト
   */
  async registerAdmin(): Promise<void> {
    //エラー表示
    this.errorMessage = "";
    this.nameErrorMessage = false;
    this.mailErrorMessage = false;
    this.passErrorMessage = false;
    this.addressErrorMessage = false;
    this.postalCodeErrorMessage = false;
    //名前空欄チェック
    if (this.lastName === "" || this.firstName === "") {
      this.nameErrorMessage = true;
    }
    //メールアドレス空欄チェック
    if (this.mailAddress === "") {
      this.mailErrorMessage = true;
    }
    //パスワード空欄チェック
    if (this.password === "") {
      this.passErrorMessage = true;
    }
    //確認用パスワード一致チェック
    if (this.verificationPassword != this.password) {
      this.verificationPassErrorMessage = true;
    }
    //郵便番号空欄チェック
    if (this.address === "") {
      this.postalCodeErrorMessage = true;
    }

    //住所空欄チェック
    if (this.postalCode === "") {
      this.addressErrorMessage = true;
    }

    if (
      this.nameErrorMessage ||
      this.mailErrorMessage ||
      this.passErrorMessage ||
      this.verificationPassErrorMessage ||
      this.postalCodeErrorMessage ||
      this.addressErrorMessage
    ) {
      return;
    }
    // 管理者登録処理
    const response = await axios.post(`${config.EMP_WEBAPI_URL}/insert`, {
      name: this.lastName + " " + this.firstName,
      mailAddress: this.mailAddress,
      password: this.password,
    });
    console.dir("response:" + JSON.stringify(response));

    if (response.data.status === "success") {
      this.$router.push("/loginAdmin");
    } else {
      this.errorMessage = "登録に失敗しました(" + response.data.message + ")";
    }
  }

  /**
   * 郵便番号から住所を取得.
   */

  //終わり
}
</script>

<style scoped>
.register-page {
  width: 600px;
}
.error {
  color: red;
}
.postbtn {
  width: 100px;
  font-size: 10px;
}
.postCode {
  display: flex;
}
</style>
