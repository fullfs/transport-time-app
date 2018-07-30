<template>
<div class="container">
    <h1 class="header">Узнай расстояние</h1>
    
    <div class="form">
        <div class="form-row" v-for="(row, index) in form" :key="index">
            <div class="form-row__item _add">
                <el-button 
                    v-if="index === 0" 
                    type="success" 
                    icon="el-icon-plus" 
                    circle
                    @click="addRow"
                    title="Добавить строку"
                    :disabled="form.length > 9"
                ></el-button>
                &nbsp;
            </div>
            <div class="form-row__item _field">
                <el-input :placeholder="getFieldPlaceholder(index, 0)" v-model="row.from"></el-input>
            </div>
            <div class="form-row__item _field">
                <el-input :placeholder="getFieldPlaceholder(index, 1)" v-model="row.to"></el-input>
            </div>
            <div class="form-row__item _controls">
                <el-button type="primary" round @click="getDistance(row)">Рассчитать</el-button>
                <el-button type="warning" round @click="clearRow(row)">Очистить</el-button>
            </div>
        </div>
    </div>

    <h1 class="header">Журнал запросов</h1>
    <div v-if="!logs.length">Запросов не было</div>
    <ul class="logs">
        <li class="logs__item" v-for="(log, index) in logs" :key="index" :class="{_error: log.error}">
            [{{log.dateString}}] {{log.error ? log.message : `${log.from} - ${log.to} = ${log.distance}`}}
        </li>
    </ul>
</div>
</template>

<script>
import get from 'lodash.get';

export default {
    data() {
        this.google = null;
        return {
            alphabet: 'АБВГДЕЁЖЗИЙКЛМНОПРСТУФХЦЧШЩЪЫЬЭЮЯ',
            form: [
                {
                    from: '',
                    to: ''
                }
            ],

            logs: []
        }
    },

    mounted() {
        // Работает только на FE, поэтому тянем в mounted через require
        const GoogleMapsLoader = require('google-maps');
        GoogleMapsLoader.load((google) => {
            this.google = google;
        });
    },
 
    methods: {
        addRow() {
            this.form.push({
                from: '',
                to: ''
            });
        },

        clearRow(row) {
            row.from = '';
            row.to = '';
        },

        getFieldPlaceholder(index, addition) {
            const base = index * 2;
            return `Пункт ${this.alphabet[base + addition]}`;
        },

        getDistance(row) {
            const service = new this.google.maps.DistanceMatrixService();
            const request =  service.getDistanceMatrix(
            {
                origins: [row.from],
                destinations: [row.to],
                travelMode: 'DRIVING'
            }, (data, status) => {
                const distance = get(data, 'rows.0.elements.0.distance')

                const today = new Date();
                const month = today.getMonth() + 1;
                const day = today.getDate();
                const hour = today.getHours();
                const minutes = today.getMinutes();
                const dateString = `${month}/${day} ${hour}:${minutes}`;
                
                if (distance) {
                    this.logs.push({
                        ...row,
                        dateString,
                        distance: distance.text,
                        error: false
                    });
                } else {
                    this.logs.push({
                        ...row,
                        dateString,
                        distance: '',
                        error: true,
                        message: 'Не удалось определить расстояние'
                    });
                }
            });
        }
    }
}
</script>

<style>
body {
    background: #f1f1f1;
}
.container {
    width: 820px;
    padding: 16px 20px 20px;
    margin: 0 auto;
    background: #fff;
    border-radius: 4px;
    font-family: Arial, Helvetica, sans-serif;
}

.header {
    font-size: 24px;
    line-height: 30px;
    margin: 0 0 18px;
}

.form {
    margin-bottom: 32px;
}

.form-row {
    display: flex;
    margin-bottom: 12px;
}

.form-row__item {
    padding-right: 12px;
}

.form-row__item._add {
    width: 50px;
}

.form-row__item._field {
    width: 240px;
}

.form-row__item._controls {
    width: 250px;
}

@media (max-width: 768px) {
    .container {
        width: 400px;
    }

    .form-row {
        display: block;
    }

    .form-row__item {
        margin-bottom: 12px;
    }
}

.logs {
    list-style: none;
    margin: 0;
    padding: 0;
}

.logs__item {
    font-size: 14px;
    line-height: 18px;
    margin-bottom: 6px;
}

.logs__item._error {
    color: #e6a23c;
}
</style>
