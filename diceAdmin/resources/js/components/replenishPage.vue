<template>
    <section class="replenish">
        <div class="date">
            <input class="date_picker date_startDate" type="date" :max="startDate" v-model="startDate"> -
            <input class="date_picker date_endDate" type="date" :min="startDate" :max="today" v-model="endDate">
        </div>
        <div class="search_section">
            <div class="search">
                <h5>Тип платёжки</h5>
                <select v-model="paymentSystemId" disabled>
                    <option>Не выбран</option>
                    <option>Неигрок</option>
                </select>
            </div>
            <div class="search">
                <h5>Статус</h5>
                <select v-model="paymentStatus">
                    <option :value="-1">Не выбран</option>
                    <option :value="0">Обработка</option>
                    <option :value="1">Оплачено</option>
                    <option :value="8">Отмена</option>
                </select>
            </div>
        </div>
        <div class="button_accept">
            <button @click="getReplenishButton">Применить</button>
        </div>
        <div class="replenish_history_table">
            <table>
                <thead>
                <tr>
                    <th>ID</th>
                    <th>user</th>
                    <th>date</th>
                    <th>sum</th>
                    <th>pay</th>
                    <th>Status</th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="item of replenish">
                    <td>{{ item.id }}</td>
                    <td>{{ item.userId }}</td>
                    <td>{{ item.createdAt }}</td>
                    <td>{{ item.userId }}</td>
                    <td class="qiwi"></td>
                    <td :class=" (item.status == 0 ) ? 'processing' : (item.status == 1 ) ? 'paid' : 'canceled' ">
                        {{ (item.status == 0 ) ? 'Обработка' : (item.status == 1 ) ? 'Оплачено' : 'Отмена' }}
                    </td>
                </tr>
                </tbody>
            </table>
            <div class="table_pagination" v-if="paginations > 1">
                <ul>
                    <li v-for ="(pagination, key) in paginationList">
                        <a href="#" @click="changePage"
                           :data-page="(pagination != '...') ? pagination + 1
                           : (paginationList[key-1] == 1) ? paginationList[key-1] + 1 : paginationList[key + 1] - 1 "
                           :class="(pagination + 1 == pageIndex) ? 'active' : ''"> {{ (pagination != '...') ? pagination + 1 : '...' }}</a>
                    </li>
                </ul>
            </div>
        </div>
    </section>
</template>
<script>
import {fetchRequest} from "@/fetch.js";
import pagination from "@/pagination.js";
export default {
    data() {
        return {
            replenish: null,
            paginations: null,
            paginationList: null,
            pageIndex: 1,
            startDate: null,
            endDate: null,
            today: null,
            paymentSystemId: -1,
            paymentStatus: -1,
        }
    },
    mounted() {
        this.getDate();
        this.getReplenish(this.pageIndex);
    },
    watch: {
        startDate(newValue, oldValue) {
            if (oldValue) {
                this.getReplenish(1);
            }
        },
        endDate(newValue, oldValue) {
            if (oldValue) {
                this.getReplenish(1);
            }
        },

    },
    methods: {
        async getReplenish(pageNumber) {
            let Url = "/admin/getPayments";
            let data = {
                "pagination": {
                    "pageNumber": pageNumber,
                    "pageSize": 20
                },
                // "startDate": this.startDate,
                // "endDate": this.endDate,

            };
            if (this.paymentSystemId) data = { ...data, "paymentSystemId": this.paymentSystemId, };
            if (this.paymentStatus >= 0) data = { ...data, "paymentStatus": this.paymentStatus };
            let payment = await fetchRequest(Url, data, localStorage.getItem('token'));
            this.replenish = payment.items;
            this.paginations = payment.totalPages;
            this.pageIndex = payment.pageIndex;
            this.paginationList = pagination(this.paginations - 1, this.pageIndex);
        },
        getReplenishButton() {
            this.getReplenish(1);
        },
        changePage(e) {
            let page = e.target.dataset.page;
            this.getReplenish(page);
        },
        getDate() {
            let date = new Date();
            this.endDate = date.toISOString().split('T')[0];
            this.today = this.endDate;
            date.setDate(date.getDate() - 1);
            this.startDate = date.toISOString().split('T')[0];
            document.querySelector(".date_startDate").value = this.startDate;
            document.querySelector(".date_endDate").value = this.endDate;
        }
    }
}
</script>
