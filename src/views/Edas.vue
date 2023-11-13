<script setup>
import DataTable from 'datatables.net-vue3';
import DataTablesCore from 'datatables.net';
import axios from 'axios'
import VueApexCharts from "vue3-apexcharts";
import {ref, onMounted} from "vue"

DataTable.use(DataTablesCore);

const url='http://api_epi.redcusconorte.gob.pe/api/'
let tipo_eda=ref('')
let nom_micro_red=ref('')
let nom_estab=ref('')

let edas=ref([])
let cod_eda=''
async function get_edas(){
  try {
      const response = await axios.get(url+'edas')
      edas.value = response.data
    } catch (e) {
      error.value = e
    }
}

let micro=ref([])
let cod_mic=''
const columns=[{data:'cod_micro_red'},{data:'nom_micro_red'}]
async function get_micro_redes(){
  try {
      const response = await axios.get(url+'micro_redes_eda')
      micro.value = response.data
    } catch (e) {
      error.value = e
    }
}

let estabs=ref([])
let cod_est=''
async function get_eess(){
  try {
      const response = await axios.get(url+'estabs_eda/'+cod_mic)
      estabs.value = response.data
    } catch (e) {
      error.value = e
    }
}

function get_tipo_eda(){
  let cmb_tipo_eda=document.getElementById('edas');
  tipo_eda=cmb_tipo_eda.options[cmb_tipo_eda.selectedIndex].text;
}

function get_nom_mic(){
  let cmb_mic=document.getElementById('micro_red');
  nom_micro_red=cmb_mic.options[cmb_mic.selectedIndex].text;
}

function get_nom_est(){
  let cmb_est=document.getElementById('eess');
  nom_estab=cmb_est.options[cmb_est.selectedIndex].text;
}


//totales edas
let totales=ref([])
let tot_eda=ref(0)
let tot_disent=ref(0)
let tot_eda_men=ref(0)
async function get_totales_edas(){
  try {
      const response = await axios.get(url+'totales_edas')
      totales.value = response.data
      if(totales.value.length>0){
        totales.value.forEach(function(num){
          switch(num['cod_eda']){
            case 1:tot_eda=num['total'];break;
            case 2:tot_disent=num['total'];break;
            case 3:tot_eda_men=num['total'];break;
          }
        });
      }
    } catch (e) {
      error.value = e
    }
}

//grafico edas por eess
let options_eda_eess=ref([])
let series_eda_eess=ref([])
let casos_edas=ref([])
let semana=ref([])
let casos=ref([])

let nom_eess=ref('')
let tiene_datos=ref(false)
async function get_casos_micro_estab_eda(){
  try {
      const response = await axios.get(url+'casos_micro_estab_eda/'+cod_mic+'/'+cod_est+'/'+cod_eda)
      casos_edas.value = response.data
      if(casos_edas.value.length>0){
        tiene_datos=true;
        let i;
        for(i=0;i<52;i++){
          casos.value[i]=0;
          semana.value[i]=i+1;
        }
        casos_edas.value.forEach(function(num){
          casos.value[num['semana']-1]=num['casos'];
          nom_eess=num['nom_eess'];
        });
      }else{
        console.log('dentro de false')
        semana.value=[]
        casos.value=[0]
        tiene_datos=false;
      }
      console.log('tiene datos:',tiene_datos);
      
    } catch (e) {
      error.value = e
    }

    options_eda_eess= {
      chart:{
        type:'line',
        id:'iras por semanas',
        dropShadow: {
            enabled: true,
            color: '#000',
            top: 8,
            left: 3,
            blur: 7,
            opacity: 0.7
          },
      },
      stroke: {
      curve: 'smooth',
      width:6,
      },
      colors: ['#F7491F'],
      dataLabels: {
              enabled: true,
      },
      
      markers: {
          size: 3,
      },
      xaxis: {
        categories: semana.value,
        title: {
          text: 'Semanas Epidemiologicas '
        },
      },
      title: {
            text: tipo_eda +' - '+ nom_eess,
            align: 'left'
          },
      yaxis: {
        title: {
          text: 'Nº de Casos'
        },
      },
    }
    series_eda_eess= [
        {
          data: casos.value,
        },
    ]
}

let options_eda_tot_eess=ref([])
let series_eda_tot_eess=ref([])
let tot_eess=ref([])
let casos_tot_eess=ref([])
let nom_tot_eess=ref([])
let nom_tot_micro=''
const columns_tot_eess=[{data:'nom_eess'},{data:'total',className:'derecha'}]
const opciones={responsive:true, pageLength:15, autowidth:false, language:{search:'Buscar', decimal:'.',thousands:',', lengthMenu:'Mostrando _MENU_ registros por pagina', zeroRecords:'No hay registros que mostrar', info:'Mostrando _START_ a _END_ de _TOTAL_ registros', infoEmpty:'Mostrando registros del 0 al 0 de un total de 0 registros',
infoFiltered:'(filtrado de un total de _MAX_ registros)',paginate:{firts:'Primero', previous:'Anterior', next:'Siguiente', last:'Ultimo'}} }
async function get_totales_edas_eess(){
  try {
      const response = await axios.get(url+'totales_edas_eess/'+cod_mic+'/'+cod_eda)
      tot_eess.value = response.data
      if(tot_eess.value.length>0){
        console.log('dentro de totales_eess:',tot_eess.value)
        tiene_datos=true;
        casos_tot_eess.value=tot_eess.value.map((element)=>{
          return parseInt(element.total);
        })
        nom_tot_eess.value=tot_eess.value.map((element)=>{
          nom_tot_micro=element.nom_micro_red
          return element.nom_eess;
        })
      }else{
        nom_tot_eess.value=[]
        casos_tot_eess.value=[0]
        tiene_datos=false;
      }

    } catch (e) {
      error.value = e
    }
    options_eda_tot_eess= {
      chart:{
        type:'bar',
        id:'iras por semanas',
        dropShadow: {
            enabled: true,
            color: '#000',
            top: 2,
            left: 1,
            blur: 4,
            opacity: 0.5
          },
      },
      title: {
              text: tipo_eda +' - '+ 'Micro Red '+nom_tot_micro,
              align: 'center'
            },
      plotOptions: {
              bar: {
                borderRadius: 8,
                horizontal: true,
              }
            },
        xaxis: {
          categories: nom_tot_eess.value,
          title: {
            text: 'Nro. de Casos'
          },
        },
        yaxis: {
          title: {
            text: 'Establecimientos'
          },
        },
    }
    series_eda_tot_eess= [
        {
          data: casos_tot_eess.value,
        },
    ]

}


let tot_edas_micro=ref([])
let casos_tot_edas_micro=ref([])
let semana_tot_edas_micro=ref([])
let options_tot_edas_micro=ref([])
let series_tot_edas_micro=ref([])
let nom_edas_micro=ref('')
let tip_eda_micro=ref('')
let tiene_datos_edas_micro=ref(false)
async function get_totales_edas_micro(){
  try {
      const response = await axios.get(url+'totales_edas_micro/'+cod_mic+'/'+cod_eda)
      tot_edas_micro.value = response.data
      console.log('tot_edas_micro:',tot_edas_micro.value)
      if(tot_edas_micro.value.length>0){
        tiene_datos_edas_micro=true;
        let i;
        for(i=0;i<52;i++){
          casos_tot_edas_micro.value[i]=0;
          semana_tot_edas_micro.value[i]=i+1;
        }
        tot_edas_micro.value.forEach(function(num){
          casos_tot_edas_micro.value[num['semana']-1]=num['casos'];
          nom_edas_micro=num['nom_micro_red'];
          tip_eda_micro=num['nom_ira'];
        });
      }else{
        semana_tot_edas_micro.value=[]
        casos_tot_edas_micro.value=[0]
        tiene_datos_edas_micro=false;
      }


    } catch (e) {
      error.value = e
    }

    options_tot_edas_micro= {
      chart:{
        type:'line',
        id:'iras por semanas',
        dropShadow: {
            enabled: true,
            color: '#022976',
            top: -1,
            left: 3,
            blur: 8,
            opacity: 0.5
          },
      },
      stroke: {
      curve: 'smooth',
      width:6,
      },
      dataLabels: {
              enabled: true,
      },
      markers: {
          size: 3,
      },
      colors: ['#064AD3'],
      xaxis: {
        categories: semana_tot_edas_micro.value,
        title: {
          text: 'Semanas Epidemiologicas'
        },
      },
      yaxis: {
        title: {
          text: 'Nro. de Casos'
        },
      },
    }
    series_tot_edas_micro= [
        {
          data: casos_tot_edas_micro.value,
        },
    ]
}

let options_tot_mic_eda=ref([])
let series_tot_mic_eda=ref([])

const columns_tot_micro_eda=[{data:'nom_micro_red'},{data:'total',className:'derecha'}]
let tot_micro_edas=ref([])
let tot_mic_eda=ref([])
let nom_mic_eda=ref([])
async function get_totales_micro_eda(){
  try {
    console.log('url micro eda:',url+'totales_micro_eda/'+cod_eda)
      const response = await axios.get(url+'totales_micro_eda/'+cod_eda)
      tot_micro_edas.value = response.data
      tot_mic_eda.value=[]
      nom_mic_eda.value=[0]
      console.log('micro edas:',tot_micro_edas.value)
      if(tot_micro_edas.value.length>0){
        console.log('dentro de tot micro');
        let i=0;
        tot_micro_edas.value.forEach(function(num){
          tot_mic_eda.value[i]=num['total'];
          nom_mic_eda.value[i]=num['nom_micro_red'];
          i++;
        });
        

      }else{
        tot_mic_eda.value=[]
        nom_mic_eda.value=[0]
      }
      console.log('tot_mic_eda:',tot_mic_eda.value);
      console.log('nom_mic_eda:',nom_mic_eda.value);
  } catch (e) {
    error.value = e
  }

  options_tot_mic_eda= {
      chart:{
        type:'bar',
        id:'edas por semanas',
        dropShadow: {
          enabled: true,
          color: '#022976',
          top: -1,
          left: 1,
          blur: 3,
          opacity: 0.5
        }
      },
      colors: ['#0B7802'],
      title: {
              text: tipo_eda +' - '+ 'Por Micro Red',
              align: 'center'
            },
      plotOptions: {
              bar: {
                borderRadius: 4,
                horizontal: true,
              }
            },
      
      xaxis: {
        categories: nom_mic_eda.value,
        title: {
          text: 'Nro. de Casos'
        },
      },
      yaxis: {
        title: {
          text: 'Micro Redes'
        },
      },
    }
    series_tot_mic_eda= [
        {
          data: tot_mic_eda.value,
        },
    ]
}



onMounted(()=>{
  get_totales_edas()
  get_micro_redes()
  get_edas()


})

</script>

<template>
  <main>
    <div class="container-fluid mb-5">
      <h1 class="tit1 text-center">Seguimiento de EDAS</h1>
      <div class="row">
        <div class="col-md-2">
          <!--edas-->
          <label for="edas" class="col-form-label etiq"><h4 class="tit1">Tipo de EDAS</h4></label>
          <select @change="get_tipo_eda(), get_casos_micro_estab_eda(), get_totales_edas_eess(), get_totales_edas_micro(), get_totales_micro_eda()" v-model="cod_eda" id="edas" class="form-select" multiple size="3">
            <option v-for="eda in edas" :value="eda.cod_eda">
              {{ eda.nom_eda }}
            </option>
          </select>
          <!--micro redes-->
          <label for="micro_red" class="col-form-label etiq"><h4 class="tit1">Micro Red</h4></label>
          <select @change="get_nom_mic(), get_eess(), get_casos_micro_estab_eda(), get_totales_edas_eess(), get_totales_edas_micro(), get_totales_micro_eda()" v-model="cod_mic" id="micro_red" class="form-select">
            <option value="" disabled selected>Seleccione una Micro Red</option>
            <option v-for="mic in micro" :value="mic.cod_micro_red">
              {{ mic.nom_micro_red }}
            </option>
          </select>
          <!--establecimientos-->
          <label for="eess" class="col-form-label etiq"><h4 class="tit1">Establecimientos</h4></label>
          <select @change="get_nom_est(), get_casos_micro_estab_eda()" v-model="cod_est" id="eess" class="form-select" multiple size="15">
            <option v-for="est in estabs" :value="est.cod_eess">
              {{ est.nom_eess }}
            </option>
          </select>
          <p style="color:white;visibility: hidden;">
            casos:<p> {{ casos }}</p>
            totales micro:{{ casos_tot_edas_micro }}
            {{ tot_micro_edas }}
          </p>
          </div>


          <div class="col-md-10">
            <div class="row">
              <h2 class="tit1">Totales RED CUSCO NORTE</h2>  
              <div class="col-md-4">
                <div class="card border-white mb-1 caja bg-primary" style="max-width: 28rem;">
                  <div class="card-header text-center text-white"><h4 class="tit1">EDAS</h4></div>
                    <div class="card-body text-white">
                      <p class="card-text bg-transparent border-white titulo3 derecha"><h2 class="tit2 text-center text-white">{{tot_eda}}</h2></p>
                    </div>
                </div>
              </div>
              <div class="col-md-4">
                <div class="card border-white mb-1 caja bg-danger" style="max-width: 28rem;">
                  <div class="card-header text-center text-white"><h4 class="tit1">DISENTERIA</h4></div>
                    <div class="card-body text-white">
                      <p class="card-text bg-transparent border-white titulo3 derecha"><h2 class="tit2 text-center text-white">{{tot_disent}}</h2></p>
                    </div>
                </div>
              </div>
              <div class="col-md-4">
                <div class="card border-white mb-1 caja bg-success" style="max-width: 28rem;">
                  <div class="card-header text-center text-white"><h4 class="tit1">EDAS MENORES.5A</h4></div>
                    <div class="card-body text-primary">
                      <p class="card-text bg-transparent border-primary titulo3 derecha"><h2 class="tit2 text-center text-white">{{tot_eda_men}}</h2></p>
                    </div>
                </div>
              </div>
            </div>

            <div class="row">
              <div class="col-md-12">
                <h2 class="tit1">Seguimiento de casos por semana/establecimiento</h2>
                <div class="caja p-3">
                  <div v-if="tiene_datos===true" class="col-auto text-center">
                    <h4 class="tit2">Casos de {{ tipo_eda }} {{nom_eess}}</h4>
                  </div>
                  <div v-else class="col-auto text-center">
                    <h4 class="tit2">Sin Casos Reportados</h4>
                  </div>
                  <VueApexCharts width="100%" height=400px
                    :options='options_eda_eess'
                    :series='series_eda_eess'
                    />
                </div>
              </div>
            </div>

            <div class="row">
              <h1 class="tit1">Totales por Establecimiento</h1>
              <div class="col-md-5">
                <div class="caja p-3">
                  <DataTable :data="tot_eess" :columns="columns_tot_eess" :options="opciones" class="display hover cell-border table compact table-striped">
                    <thead>
                        <tr>
                            <th style="width: 80%;">Establecimiento</th>
                            <th style="width: 20%;">Casos</th>
                        </tr>
                    </thead>
                  </DataTable>
                </div>
              </div>

              <div class="col-md-7">
                <div class="caja p-3">
                  <VueApexCharts width="100%" height=600px
                  :options='options_eda_tot_eess'
                  :series='series_eda_tot_eess'
                  />
                </div>
              </div>
            </div>

            <h2 class="tit1 mt-2">Seguimiento de casos por semana/Micro Red</h2>  
          <div class="row">
            <div class="col-md-12">
              <div class="caja p-3">
                <div v-if="tiene_datos_edas_micro===true" class="col-auto text-center">
                  <h4 class="tit2">Casos de {{ tipo_eda }} Micro Red {{nom_edas_micro}}</h4>
                </div>
                <div v-if="tiene_datos_edas_micro===false" class="col-auto text-center">
                  <h4 class="tit2">Sin Casos Reportados</h4>
                </div>
                <VueApexCharts width="100%" height=400px
                :options='options_tot_edas_micro'
                :series='series_tot_edas_micro'
                />
              </div>
            </div>
          </div>

          <div class="row">
            <h1 class="tit1">Totales por Micro Red</h1>
            <div class="col-md-5">
              <div class="caja p-3">
                <DataTable :data="tot_micro_edas" :columns="columns_tot_micro_eda" :options="opciones" class="display hover cell-border table compact table-striped">
                  <thead>
                      <tr>
                          <th style="width: 80%;">Micro Red</th>
                          <th style="width: 20%;">Casos</th>
                      </tr>
                  </thead>
                </DataTable>
              </div>
            </div>

            <div class="col-md-7">
              <div class="caja p-3">
                <VueApexCharts width="100%" height=500px
                :options='options_tot_mic_eda'
                :series='series_tot_mic_eda'
                />
              </div>
            </div>
          </div>


          </div>
      </div>
    </div>
  </main>
</template>

<style>
.tit1{
  font-weight: 800;
}
.tit2{
  font-weight: 600;
  color: #0981F1;
}

 

</style>
