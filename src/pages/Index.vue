<template>
  <q-page class="">
  <div class="q-pa-md">
    <q-input
        filled
        v-model="searchInfo"
        label="搜尋過敏原類型"
      />
      <q-btn
        class="q-ma-md"
        color="secondary"
        label="搜尋"
        icon="search"
        @click="goSearch()"
      />
      <q-table
      title="以過敏原類型搜尋"
      :data="result"
      :columns="columns"
      row-key="name"
    />
  </div>
    <div class="q-pa-md">
    <q-table
      title="過敏原"
      :data="data"
      :columns="columns"
      row-key="name"
    />
    <q-btn
      class="q-mt-md"
      color="secondary"
      label="新增過敏原"
      icon="add"
      @click="addSymptom = true"
    />
    <q-dialog v-model="addSymptom" persistent>
      <q-card style="min-width: 350px">
        <h6 class="text-primary q-ma-md">新增過敏原</h6>
        <q-card-section class="q-pt-none">
          <q-input dense v-model="symptomInfo.symID" label="過敏原編號" type="number"/>
          <q-input dense v-model="symptomInfo.symName" label="過敏原名稱" />
          <q-input dense v-model="symptomInfo.symCategory" label="過敏原類型" />
        </q-card-section>

        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="取消" @click="onClose" />
          <q-btn flat label="新增" @click="add(symptomInfo.symID)" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
  </q-page>
</template>

<script>
import { getDocs, collection, getFirestore ,setDoc ,doc ,query} from "firebase/firestore";
import { and, where } from "firebase/firestore";
import app from '../components/setting/FirebaseConfig';

export default {
  name: 'PageIndex',
  data(){
      return {
      columns: [
        {
          name: 'symID',
          required: true,
          label: '過敏原編號',
          align: 'center',
          field: 'symID',
          format: val => `${val}`,
          sortable: true
        },
        { name: 'symName', align: 'center', label: '過敏原名稱', field: 'symName', sortable: true },
        { name: 'symCategory', align: 'center', label: '過敏原類型', field: 'symCategory', sortable: true },
        { name: 'symCase', align: 'center', label: '過敏原案例數', field: 'symCase', sortable: true }
      ],
      data: [],
      result: [
        {symID: '#',
        symName: '',
        symCategory: '',
        symCase: ''},
      ],
      searchInfo: '',
      addSymptom: false,
      symptomInfo: {
        symCase: 0
      }
    }
    },
    methods: {
      onClose(){
        this.addSymptom = false;
        this.symptomInfo={}
      },
      async getSymptom(){
        const db = getFirestore(app);
        const querySnapshot = await getDocs(collection(db, "symptom"));
        querySnapshot.forEach((doc) => {
          this.data.push(doc.data());
        });
        // console.log("before",this.data);
        const querySnapshot2 = await getDocs(collection(db, "patient"));
        querySnapshot2.forEach((doc) => {
            // console.log(doc.data());
            for (var j = 0; j < this.data.length; j++) {//所有過敏原
              for (var i = 0; i < doc.data().symptom.length; i++) {//一個病患有多個過敏原
                if (doc.data().symptom[i].symID == this.data[j].symID) {
                  // console.log('tag', i)
                  // console.log("病患過敏原",doc.data().symptom[i].symID);
                  // console.log(this.data[j].symID);
                  this.data[j].symCase ++;
                  // console.log("過敏原案例數",this.data[j].symCase)
                }
          }
        }
        });
        console.log("after",this.data);
      },
      async add(v){
        const db = getFirestore(app);
        await setDoc(doc(db, "symptom", v),this.symptomInfo);
        this.addSymptom = false;
        console.log("新增成功");
        this.$q.notify({
        message: '新增成功',
        color: 'secondary'
      })
      },
      async goSearch(){
        this.result = [];
        const db = getFirestore(app);
        const keyword = this.searchInfo.toLowerCase();
        const q = query(collection(db, "symptom"), where('symCategory', '==', keyword))
        console.log("000",keyword);
        try {
          const querySnapshot = await getDocs(q);
          querySnapshot.forEach((doc) => {
            console.log(doc.data());
            this.result.push(doc.data());
            console.log("success");
          });
        } catch (error) {
          console.error("Error getting documents: ", error);
        }
      }
    },
    mounted(){
      this.getSymptom();
    }
}
</script>
