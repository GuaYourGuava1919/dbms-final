<template>
  <q-page class="">
    <div class="q-pa-md">
    <q-table
      title="病患"
      :data="data"
      :columns="columns"
      row-key="name"
    >
    <template v-slot:body-cell-action="props">
      <q-btn color="negative" flat round  icon="delete" @click="remove(props.row)" />
      <q-btn color="secondary" flat round  icon="edit" @click="editRow()" />
    </template>
    </q-table>
    <q-btn
      class="q-mt-md"
      color="secondary"
      label="新增病患"
      icon="add"
      @click="addPatient = true"
    />
    <q-dialog v-model="addPatient" persistent>
      <q-card style="min-width: 350px">
        <h6 class="text-primary q-ma-md">新增病患</h6>
        <q-card-section class="q-pt-none">
          <q-input dense v-model="patientInfo.patientID" label="病患編號" type="number"/>
          <q-input dense v-model="patientInfo.patientName" label="病患名稱" />
          <q-input dense v-model="patientInfo.patientBirth" label="病患出生年月日" type="date"/>
          <q-input dense v-model="patientInfo.patientAdd" label="病患住址" />
          <q-select
            dense
            v-model="patientInfo.symptom"
            :options="options"
            option-label="symName"
            option-value="symID"
            label="過敏原"
            multiple
          />

        </q-card-section>

        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="取消" @click="onClose"/>
          <q-btn flat label="新增" @click="add(patientInfo.patientID)" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
  </q-page>
</template>

<script>
import { getDocs, collection, getFirestore,setDoc,doc ,deleteDoc} from "firebase/firestore";
import app from '../components/setting/FirebaseConfig';
import moment from 'moment';

export default {
  name: 'PageIndex',
  data(){
      return {
      columns: [
        {
          name: 'patientID',
          required: true,
          label: '病患編號',
          align: 'center',
          field: 'patientID',
          format: val => `${val}`,
          sortable: true
        },
        { name: 'patientName', align: 'center', label: '病患名稱', field: 'patientName', sortable: true },
        { name: 'patientBirth', align: 'center', label: '病患出生年月日', field: 'patientBirth', sortable: true },
        { name: 'patientAdd', align: 'center', label: '病患住址', field: 'patientAdd' },
        { name: 'symptom', align: 'center', label: '過敏原', field: 'symptom' },
        {
          name: 'action',
          align: 'right',
          // label: '操作',
          field: row => row,
          format: (val, row) => {
            return {
              slotName: 'actionColumn',
              content: row,
            };
          },
        },
      ],
      data: [],
      symptoms: [],
      options: [],
      addPatient: false,
      patientInfo:{
      }
      }
    },
    methods: {
      onClose(){
        this.addPatient = false;
        this.patientInfo={}
      },
      async getPatient(){
        const db = getFirestore(app);
        const querySnapshot = await getDocs(collection(db, "patient"));
        querySnapshot.forEach((doc) => {
          //時間戳格式化
          var date = moment(new Date(doc.data().patientBirth.seconds * 1000)).format('YYYY-MM-DD');
          //symptom推入陣列
          for(var i = 0; i < doc.data().symptom.length; i++){
            this.symptoms.push(
              doc.data().symptom[i].symName
            );
          }
          this.data.push({
            patientID: doc.data().patientID,
            patientName: doc.data().patientName,
            patientBirth: date,
            patientAdd: doc.data().patientAdd,
            symptom: this.symptoms.toString(),
          });
          console.log(this.data);
          this.symptoms = [];
        });
      },
      async add(v){
        console.log(this.patientInfo);
        const db = getFirestore(app);
        await setDoc(doc(db, "patient", v),this.patientInfo);
          this.addPatient = false;
          this.patientInfo={}
          console.log("新增成功");
          this.$q.notify({
          message: '新增成功',
          color: 'secondary'
          })
      },
      async getSym(){
        const db = getFirestore(app);
        const querySnapshot = await getDocs(collection(db, "symptom"));
        querySnapshot.forEach((doc) => {
          this.options.push(doc.data());
        });
      },
      async remove(row) {
        console.log("Here",row);
        const db = getFirestore(app);
        await deleteDoc(doc(db, "patient", row.patientID));
        console.log("刪除成功");
        this.$q.notify({
        message: '刪除成功',
        color: 'accent'
        })
      },
      editRow(row) {
      // Implement logic to edit the row in the data array or Firestore if needed
      },

    },
    mounted(){
      this.getPatient();
      this.getSym();
    }
}
</script>
