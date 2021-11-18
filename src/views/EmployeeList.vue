<template>
  <div class="container">
    <!-- パンくずリスト -->
    <form>
      <span class="error" v-show="errorMessage"
        >１件もありませんでしたので全件表示します</span
      >
      <div class="employeeSearch">
        <label for="employeeSearch">従業員名検索：</label>
        <input type="text" id="employeeSearch" v-model="searchName" />
        <div class="btnPlace">
          <button
            type="button"
            v-on:click="getemployeeSearch"
            class="btn btn-large btn-register waves-effect waves-light"
          >
            検索
          </button>
        </div>
      </div>
    </form>
    <nav>
      <div class="nav-wrapper">
        <div class="col s12 teal">
          <a class="breadcrumb">従業員リスト</a>
        </div>
      </div>
    </nav>
    <div>従業員数:{{ getEmployeeCount }}人</div>
    <div class="row">
      <table class="striped">
        <thead>
          <tr>
            <th>名前</th>
            <th>入社日</th>
            <th>扶養人数</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="employee of displayEmployeeList" v-bind:key="employee.id">
            <td>
              <router-link :to="'/employeeDetail/' + employee.id">{{
                employee.name
              }}</router-link>
            </td>
            <td>{{ employee.formatHireDate }}</td>
            <td>{{ employee.dependentsCount }}人</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="btnPlace">
      <button
        type="button"
        v-for="buttonNum of pageButtonNumber"
        :key="buttonNum"
        class="btn btn-large btn-register waves-effect waves-light"
        v-show="showButton"
        v-on:click="formatDisplayEmployee(buttonNum)"
      >
        {{ buttonNum }}
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { Employee } from "@/types/employee";
/**
 * 従業員一覧を表示する画面.
 */
@Component
export default class EmployeeList extends Vue {
  // 従業員一覧
  private currentEmployeeList: Array<Employee> = [];
  //表示用従業員一覧
  private displayEmployeeList: Array<Employee> = [];
  // 従業員数
  private employeeCount = 0;
  //検索する従業員名
  private searchName = "";
  //検索に対するエラーメッセージ
  private errorMessage = false;
  //ページボタン(検索結果表示時は見えなくする)
  private showButton = true;

  /**
   * Vuexストアのアクション経由で非同期でWebAPIから従業員一覧を取得する.
   *
   * @remarks
   * Vueインスタンスが生成されたタイミングで
   * Vuexストア内のアクションを呼ぶ(ディスパッチする)。
   * ライフサイクルフックのcreatedイベント利用。
   *
   * 取得してからゲットするため、async awaitを利用している。
   */
  async created(): Promise<void> {
    await this.$store.dispatch("getEmployeeList");
    this.currentEmployeeList = this["$store"].getters.getAllEmployees;
    // 従業員一覧情報をVuexストアから取得
    // 非同期で外部APIから取得しているので、async/await使わないとGetterで取得できない
    // ページング機能実装のため最初の10件に絞り込み
    this.firstDisplayEmployee();
  }

  /**
   * 現在表示されている従業員一覧の数を返す.
   *
   * @returns 現在表示されている従業員一覧の数
   */
  get getEmployeeCount(): number {
    return this.currentEmployeeList.length;
  }

  /**
   * 従業員の総人数に合わせてボタンの数を配置する
   */
  get pageButtonNumber(): Array<number> {
    this.showButton = true;
    const array = new Array<number>();
    //終わりのボタン番号
    let lastButton = 0;
    if (this.getEmployeeCount % 10 == 0) {
      lastButton = this.getEmployeeCount / 10;
    } else {
      lastButton = Math.ceil(this.getEmployeeCount / 10);
    }
    console.log("ラストボタン：" + lastButton);
    for (let i = 1; i <= lastButton; i++) {
      array.push(i);
    }
    return array;
  }

  /**
   * 従業員を1-10人目までに搾る(初期表示用).
   */
  firstDisplayEmployee(): void {
    for (let i = 0; i < 10; i++) {
      this.displayEmployeeList.push(this.currentEmployeeList[i]);
    }
  }

  /**
   * 従業員を10人ずつに搾って表示する(ボタン押下時用).
   */
  formatDisplayEmployee(pageNum: number): void {
    this.displayEmployeeList.splice(0, this.displayEmployeeList.length);
    const BOX_COUNT = 1;
    const firstEmployee = (pageNum - BOX_COUNT) * 10;
    let lastEmployee = pageNum * 10 + BOX_COUNT;
    if (this.currentEmployeeList.length < lastEmployee) {
      lastEmployee = this.currentEmployeeList.length - BOX_COUNT;
    }
    for (let i = firstEmployee; i <= lastEmployee; i++) {
      this.displayEmployeeList.push(this.currentEmployeeList[i]);
    }
  }

  /**
   * 従業員名をあいまい検索できるメソッド.
   */
  getemployeeSearch(): void {
    //エラーメッセージと表示する従業員の初期化
    this.errorMessage = false;
    this.firstDisplayEmployee();
    this.showButton = true;

    //検索欄が空欄だった場合全件表示
    const array = new Array<Employee>();
    if (this.searchName === "") {
      return;
    }
    //検索結果に合わせて表示するために配列を作成
    this.showButton = false;
    for (const employee of this.currentEmployeeList) {
      if (employee.name.includes(this.searchName)) {
        array.push(employee);
      }
    }
    this.displayEmployeeList = array;
    //検索結果が0件だった場合、エラーを表示して全件表示
    if (array.length === 0) {
      this.errorMessage = true;
      this.firstDisplayEmployee();
      this.showButton = true;
    }
  }
  //script
}
</script>

<style scoped>
.searchForm {
  margin-bottom: 20px;
  width: 450px;
  margin: 0 auto;
}

.error {
  color: red;
}

.btn {
  margin: 30px;
}

.btnPlace {
  text-align: center;
}

.employeeSearch {
  margin-bottom: 30px;
}
</style>
