<template>
  <div id="app">
    <b-modal v-model="modal" hide-footer>
      <div>Email: {{ current.user ? current.user.name : '' }}</div>
      <div>ФИО: {{ current.user ? current.user.email : '' }}</div>
      <h3>Описание</h3>
      {{ desc.title }}
      <div style="display: flex; flex-direction: row; margin-top: 10px; overflow-y: auto;margin-bottom: 15px">
        <div class="mr-3" v-for="(el,ind) in desc.images" v-bind:key="ind">
          <img style="width: 100px; height: 100px; border-radius: 10px; object-fit: cover"
               :src="'http://188.225.18.212/img/'+el" @click="openWindow('http://188.225.18.212/img/'+el)">
        </div>
      </div>
      <h3>Критерии</h3>
      <div style="margin-bottom: 15px;padding-bottom: 10px;border-bottom: 1px gray solid"
           v-for="(el,index) in current.criteria" v-bind:key="index">
        <div style="display: flex; flex-direction: row; justify-content: space-between">
          <div>
            {{ el.name }}
          </div>
          <div>
            <input :disabled="current.status==='accepted'" placeholder="оцените 1-5" type="number" v-model="el.rating"/>
          </div>

        </div>
        <div style="display: flex; flex-direction: row; margin-top: 10px; overflow-y: auto">
          <div class="mr-3" v-for="(el,ind) in el.images" v-bind:key="ind">
            <img style="width: 100px; height: 100px; border-radius: 10px;object-fit: cover"
                 :src="'http://188.225.18.212/img/'+el" @click="openWindow('http://188.225.18.212/img/'+el)">
          </div>
        </div>

      </div>
      <div v-if="current.status!=='accepted'">
        Ответ
        <b-textarea class="mb-2" v-model="message" max-rows="6" rows="6"/>
        <b-button class="mr-2" variant="success" @click="accept">Принять</b-button>
        <b-button class="mr-2" variant="warning" @click="back">На доработку</b-button>
        <b-button class="mr-2" variant="danger" @click="reject">Отклонить</b-button>
      </div>
      <div v-else class="text-success">
        Принят
      </div>

    </b-modal>
    <b-container>
      <b-card>


        <b-table responsive :fields="fields" :items="data" @row-clicked="openModal" class="text-left">
          <template v-slot:cell(user)="{item}">
            {{ item.user.name }}
          </template>

          <template v-slot:cell(date_start)="{item}">
            {{ yyyymmdd(item.date_start) }}

          </template>
        </b-table>
      </b-card>
      <b-row class="mt-3">
        <b-col>
          <b-card class="text-left">
            <div v-for="(el,ind) in checklist" v-bind:key="ind">
              <h5 @click="el.open=!el.open" style="cursor: pointer">{{ el.title }}</h5>
              <b-collapse v-model="el.open">
                <div v-for="(box,index) in el.boxes" v-bind:key="index" class="mt-1 checklist"
                     style="display: flex; flex-direction: row">
                  <div>
                    · {{ box.label }}
                  </div>
                  <div @click="removeCheckList(ind,index)" class="ml-auto pr-2" style="cursor: pointer">
                    X
                  </div>
                </div>
                <div class="mt-3 mb-3" style="width: 50%;display: flex;flex-direction: row">
                  <b-input v-model="el.new" class="mr-2"/>
                  <b-button @click="addChecklist(ind)">Добавить</b-button>
                </div>

              </b-collapse>
            </div>
            <b-button @click="saveChecklist" variant="success" class="mt-4">Сохранить</b-button>
          </b-card>
        </b-col>
      </b-row>

      <b-row align-content="stretch">
        <b-col class="text-left">
          <b-card class="mt-3">

            <h5>email</h5>
            Заголовок
            <b-input v-model="mail.email_theme"/>
            <div class="mt-3">Текст</div>
            <b-textarea v-model="mail.email_text" max-rows="8" rows="8" class="mt-2"/>
            <div class="mt-3">Футер</div>
            <b-textarea v-model="mail.email_footer" max-rows="5" rows="5" class="mt-2"/>
            <b-button @click="setEmail" class="mt-2 ml-auto" variant="success">Сохранить</b-button>
          </b-card>
        </b-col>
        <b-col class="text-left">
          <b-card class="mt-3">
            <h5>Инфо</h5>
            Заголовок
            <b-input v-model="info.title"/>
            <div class="mt-3"> Описание</div>
            <b-textarea v-model="info.text" max-rows="8" rows="8" class="mt-2"/>
            <div class="mt-3">Изображения</div>
            <input type="file" id="file" ref="file">
            <b-button @click="handleFileUpload">Добавить</b-button>

            <div class="flex-container" style="display: flex; flex-direction: row; margin-top: 10px;">
              <div v-if="loading"
                   style="width: 100px; height: 100px; display: flex; justify-content: center; align-items: center">
                <b-spinner/>
              </div>

              <div class="mr-3" style="position: relative" v-for="(el,ind) in info.icons" v-bind:key="ind">
                <div style="position: absolute; cursor: pointer" @click="removeImg(ind)">X</div>
                <img style="width: 100px; height: 100px; border-radius: 10px; object-fit: cover"
                     :src="el" @click="openWindow('http://188.225.18.212/img/'+el)">
              </div>


            </div>

            <b-button @click="saveInfo" class="mt-2 ml-auto" variant="success">Сохранить</b-button>
          </b-card>
        </b-col>
      </b-row>

    </b-container>
  </div>
</template>

<script>

export default {
  name: 'App',
  components: {},
  data() {
    return {
      data: [],
      modal: false,
      message: '',
      mail: {
        email_text: "",
        email_theme: "",
        email_footer: ""
      },
      info: {
        title: "",
        text: "",
        icons: []
      },
      loading: false,
      current: {},
      checklist: [],
      fields: [
        {key: 'name', label: 'Название'},
        {key: 'user', label: 'Пользователь'},
        {key: 'address', label: 'Адрес'},
        {key: 'status', label: 'Статус'},
        {key: 'date_start', label: 'Дата'},
      ],
    }
  },
  async mounted() {
    let {data} = await this.$axios.get('http://188.225.18.212/event/-/admin');
    console.log(data)
    this.data = data;
    this.getEmail();
    this.getInfo();
    this.getChecklist();
  },
  methods: {
    openModal(row) {
      this.modal = true;
      this.current = row;
      this.message = '';
    },
    async accept() {
      console.log(this.current)
      let {data} = await this.$axios.put('http://188.225.18.212/event/' + this.current._id + '/admin', {
        ...this.current,
        status: 'accepted',
        message:this.message
      });
      console.log(data)
    },
    async reject() {
      console.log(this.current)
      let {data} = await this.$axios.put('http://188.225.18.212/event/' + this.current._id + '/admin', {
        ...this.current,
        status: 'rejected',
        message:this.message
      });
      console.log(data)
    },
    async back() {
      console.log(this.current)
      let {data} = await this.$axios.put('http://188.225.18.212/event/' + this.current._id + '/admin', {
        ...this.current,
        status: 'draft',
        message:this.message
      });
      console.log(data)
    },
    async getEmail() {
      console.log(this.current)
      let {data} = await this.$axios.get('http://188.225.18.212/checklist/email/body');
      this.mail = data;
    },
    async setEmail() {
      console.log(this.current)
      await this.$axios.post('http://188.225.18.212/checklist/email/body', this.mail);
    },

    async getInfo() {
      const {data} = await this.$axios.get('http://188.225.18.212/info');
      this.info = data[0];
    },
    async getChecklist() {
      const {data} = await this.$axios.get('http://188.225.18.212/checklist');
      this.checklist = data;
    },
    yyyymmdd(dateIn) {
      if (!dateIn) return '-';
      const date = new Date(dateIn);

      return date.getFullYear() + '-' + ('0' + (date.getMonth() + 1)).slice(-2) + '-' + ('0' + date.getDate()).slice(-2)
      // Leading zeros for mm and dd
    },
    openWindow(url) {
      console.log(url);
      window.open(url);
    },
    addChecklist(index) {
      this.checklist[index].boxes.push({label: this.checklist[index].new})
      delete this.checklist[index].new;
    },
    removeCheckList(i, j) {
      this.checklist[i].boxes.splice(j, 1);

    },
    async handleFileUpload() {
      this.loading = true;
      try {
        const file = this.$refs.file.files[0];
        console.log(this.$refs.file.files);
        let formData = new FormData();
        formData.append('image', file);
        const {data} = await this.$axios.post('http://188.225.18.212/img/upload', formData, {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        });
        this.info.icons.unshift('http://188.225.18.212/img/' + data.filename);
        console.log(data);
      } finally {
        this.loading = false;
      }

    },
    async saveInfo() {
      await this.$axios.post('http://188.225.18.212/info', this.info,);
    },
    async saveChecklist() {
      await this.$axios.post('http://188.225.18.212/checklist', {checklist: this.checklist},);
    },
    removeImg(ind) {
      this.info.icons.splice(ind, 1)
    }

  },
  computed: {
    desc() {
      return this.current.description || {}
    }
  }

}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  background: #f3f3f3;
}

html {
  background: #f3f3f3;
}

.flex-container {
  display: flex;
  flex-wrap: nowrap;
  white-space: nowrap;
  overflow-x: auto;
  max-width: 30vw;
}

.flex-container .item:first-child {
  margin-left: auto;
}

.flex-container .item:last-child {
  margin-right: auto;
}

.checklist:hover {
  background: #f5f5f5;
}
</style>
