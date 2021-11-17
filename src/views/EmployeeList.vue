<template>
  <div class="container">
    <!-- パンくずリスト -->
    <form>
      <span class="error" v-show="errorMessage">検索結果がありません</span>
      <div class="employeeSearch">
        <label for="employeeSearch">従業員名検索：</label>
        <input type="text" id="employeeSearch" v-model="searchName" />
        <button type="button" v-on:click="getemployeeSearch">検索</button>
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
          <tr v-for="employee of currentEmployeeList" v-bind:key="employee.id">
            <td>
              <router-link :to="'/employeeDetail/' + employee.id">{{
                employee.name
              }}</router-link>
            </td>
            <td>{{ employee.hireDate }}</td>
            <td>{{ employee.dependentsCount }}人</td>
          </tr>
        </tbody>
      </table>
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
  // 従業員数
  private employeeCount = 0;
  //検索する従業員名
  private searchName = "";
  //検索に対するエラーメッセージ
  private errorMessage = false;

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

    // 従業員一覧情報をVuexストアから取得
    // 非同期で外部APIから取得しているので、async/await使わないとGetterで取得できない
    // ページング機能実装のため最初の10件に絞り込み
    this.currentEmployeeList = this.$store.getters.getAllEmployees;
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
   * 従業員名をあいまい検索できるメソッド.
   */
  getemployeeSearch(): void {
    this.errorMessage = false;
    this.currentEmployeeList = this.$store.getters.getAllEmployees;
    const array = new Array<Employee>();
    if (this.searchName === "") {
      return;
    }
    for (const employee of this.currentEmployeeList) {
      this.currentEmployeeList;
      if (employee.name.includes(this.searchName)) {
        console.dir("取得した情報：" + JSON.stringify(employee));
        array.push(employee);
      }
    }
    this.currentEmployeeList = array;
    if (array.length === 0) {
      this.errorMessage = true;
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

.searchBtn {
  display: block;
  width: 150px;
  margin: 0 auto;
}

.employeeSearch {
  margin-bottom: 30px;
}
</style>
