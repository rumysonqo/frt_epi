<script setup>
import DataTable from 'datatables.net-vue3';
import DataTablesCore from 'datatables.net';
import axios from 'axios'
import VueApexCharts from "vue3-apexcharts";
import {ref, onMounted} from "vue"


DataTable.use(DataTablesCore);

const url='http://api_epi.redcusconorte.gob.pe/api/'
let tipo_ira=ref('')
let nom_micro_red=ref('')
let nom_estab=ref('')


let iras=ref([])
let cod_ira=''
async function get_iras(){
  try {
      const response = await axios.get(url+'iras')
      iras.value = response.data
    } catch (e) {
      error.value = e
    }
}

function get_tipo_ira(){
  let cmb_tipo_ira=document.getElementById('iras');
  tipo_ira=cmb_tipo_ira.options[cmb_tipo_ira.selectedIndex].text;
}

function get_nom_mic(){
  let cmb_mic=document.getElementById('micro_red');
  nom_micro_red=cmb_mic.options[cmb_mic.selectedIndex].text;
}

function get_nom_est(){
  let cmb_est=document.getElementById('eess');
  nom_estab=cmb_est.options[cmb_est.selectedIndex].text;
}


let micro=ref([])
let cod_mic=''
const columns=[{data:'cod_micro_red'},{data:'nom_micro_red'}]
async function get_micro_redes(){
  try {
      const response = await axios.get(url+'micro_redes')
      micro.value = response.data
      console.log('datos:',micro.value)
    } catch (e) {
      error.value = e
    }
}

let totales=ref([])
let tot_ira=ref(0)
let tot_neumonia=ref(0)
let tot_sob_men_2=ref(0)
let tot_sob_2_4=ref(0)
let tot_muertes=ref(0)
let tot_neumonia_adulto=ref(0)
async function get_totales_iras(){
  try {
      const response = await axios.get(url+'totales_iras')
      totales.value = response.data
      if(totales.value.length>0){
        totales.value.forEach(function(num){
          switch(num['cod_ira']){
            case 1:tot_ira=num['total'];break;
            case 2:tot_neumonia=num['total'];break;
            case 3:tot_sob_men_2=num['total'];break;
            case 4:tot_sob_2_4=num['total'];break;
            case 5:tot_muertes=num['total'];break;
            case 6:tot_neumonia_adulto=num['total'];break;
          }
          casos.value[num['semana']-1]=num['casos'];
          /*nom_eess=num['nom_eess'];*/
        });

      }
    } catch (e) {
      error.value = e
    }
}

let estabs=ref([])
let cod_est=''
async function get_eess(){
  try {
      const response = await axios.get(url+'estabs/'+cod_mic)
      estabs.value = response.data
      console.log('dat_gen:',estabs.value)
    } catch (e) {
      error.value = e
    }
}

let casos_iras=ref([])
let semana=ref([])
let casos=ref([])
let options=ref([])
let series=ref([])
async function get_casos_micro_estab_ira(){
  try {
      console.log('url graf1:',url+'casos_micro_estab_ira/'+cod_mic+'/'+cod_est+'/'+cod_ira)
      const response = await axios.get(url+'casos_micro_estab_ira/'+cod_mic+'/'+cod_est+'/'+cod_ira)
      casos_iras.value = response.data
      if(casos_iras.value.length>0){
        let i;
        for(i=0;i<52;i++){
          casos.value[i]=0;
          semana.value[i]=i+1;
        }
        casos_iras.value.forEach(function(num){
          casos.value[num['semana']-1]=num['casos'];
        });
      }else{
        semana.value=[]
        casos.value=[0]
      }
      
    } catch (e) {
      error.value = e
    }

    options= {
      chart:{
      type:'line',
      id:'iras por semanas',
      dropShadow: {
              enabled: true,
              color: '#022976',
              top: -1,
              left: 2,
              blur: 5,
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
      colors: ['#056DE2'],
      xaxis: {
        categories: semana.value,
        title: {
          text: 'Semanas Epidemiologicas '
        },
      },
      title: {
            text: tipo_ira +' - '+ nom_estab,
            align: 'left'
          },
      yaxis: {
        title: {
          text: 'Nº de Casos'
        },
      },
        }
        series= [
            {
              data: casos.value,
            },
        ]
}

let tot=ref([])
const columns_tot=[{data:'nom_eess'},{data:'total',className:'derecha'}]
const opciones={responsive:true, pageLength:15, autowidth:false, language:{search:'Buscar', decimal:'.',thousands:',', lengthMenu:'Mostrando _MENU_ registros por pagina', zeroRecords:'No hay registros que mostrar', info:'Mostrando _START_ a _END_ de _TOTAL_ registros', infoEmpty:'Mostrando registros del 0 al 0 de un total de 0 registros',
infoFiltered:'(filtrado de un total de _MAX_ registros)',paginate:{firts:'Primero', previous:'Anterior', next:'Siguiente', last:'Ultimo'}} }
let options_tot=ref([])
let series_tot=ref([])
let casos_tot=ref([])
let eess_tot=ref([])
let nom_tot_micro=ref('')
async function get_totales_iras_eess(){
  try {
      const response = await axios.get(url+'totales_iras_eess/'+cod_mic+'/'+cod_ira)
      tot.value = response.data
      if(tot.value.length>0){
        casos_tot.value=tot.value.map((element)=>{
          nom_tot_micro=element.nom_micro_red;
          //tipo_ira=element.nom_ira;
          return parseInt(element.total);
        })
        eess_tot.value=tot.value.map((element)=>{
            return element.nom_eess;
        })
        
      }else{
        eess_tot.value=[]
        casos_tot.value=[0]
        tiene_datos=false;
      }

    } catch (e) {
      error.value = e
    }
    options_tot= {
      chart:{
        type:'bar',
        id:'iras por semanas',
        dropShadow: {
              enabled: true,
              color: '#022976',
              top: -1,
              left: 1,
              blur: 3,
              opacity: 0.5
            },
      },
      colors: ['#056DE2'],
      title: {
              text: tipo_ira +' - '+ 'Micro Red '+nom_tot_micro,
              align: 'center'
            },
      plotOptions: {
              bar: {
                borderRadius: 8,
                horizontal: true,
              }
            },
        xaxis: {
          categories: eess_tot.value,
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
    series_tot= [
        {
          data: casos_tot,
        },
    ]
}




let tot_micro=ref([])
let casos_tot_micro=ref([])
let semana_tot_micro=ref([])
let options_tot_micro=ref([])
let series_tot_micro=ref([])


let nom_micro=ref('')
let tiene_datos_micro=ref('false')
async function get_totales_iras_micro(){
  try {
      const response = await axios.get(url+'totales_iras_micro/'+cod_mic+'/'+cod_ira)
      tot_micro.value = response.data

      if(tot_micro.value.length>0){
        let i;
        for(i=0;i<52;i++){
          casos_tot_micro.value[i]=0;
          semana_tot_micro.value[i]=i+1;
        }
        tot_micro.value.forEach(function(num){
          casos_tot_micro.value[num['semana']-1]=num['casos'];
          nom_micro=num['nom_micro_red'];
          //tipo_ira=num['nom_ira'];
        });
      }else{
        semana_tot_micro.value=[]
        casos_tot_micro.value=[0]
      }


    } catch (e) {
      error.value = e
    }

    options_tot_micro= {
      chart:{
        type:'line',
        id:'iras por semanas',
        dropShadow: {
          enabled: true,
          color: '#022976',
          top: -1,
          left: 1,
          blur: 3,
          opacity: 0.5
        }
      },
      dataLabels: {
              enabled: true,
      },
      markers: {
          size: 3,
      },
      colors: ['#0B7802'],
      title: {
              text: 'Casos de '+tipo_ira+' - Micro Red '+nom_micro_red,
              align: 'center'
            },
      
      xaxis: {
        categories: semana_tot_micro.value,
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
    series_tot_micro= [
        {
          data: casos_tot_micro.value,
        },
    ]
}

let options_tot_mic=ref([])
let series_tot_mic=ref([])

const columns_tot_micro=[{data:'nom_micro_red'},{data:'total',className:'derecha'}]
let tot_iras_micro=ref ([])
let tot_mic=ref([])
let nom_mic=ref([])
async function get_totales_micro(){
  try {
      const response = await axios.get(url+'totales_micro/'+cod_ira)
      tot_iras_micro.value = response.data
      tot_mic.value=[]
      nom_mic.value=[0]
      if(tot_iras_micro.value.length>0){
        let i=0;
        console.log('tipo ira totales micro:',tot_iras_micro.value)
        tot_iras_micro.value.forEach(function(num){
          tot_mic.value[i]=num['total'];
          nom_mic.value[i]=num['nom_micro_red'];
          i++;
        });
      }else{
        tot_mic.value=[]
        nom_mic.value=[0]
      }
  } catch (e) {
    error.value = e
  }

  options_tot_mic= {
      chart:{
        type:'bar',
        id:'iras por semanas',
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
              text: tipo_ira +' - '+ 'Por Micro Red',
              align: 'center'
            },
      plotOptions: {
              bar: {
                borderRadius: 4,
                horizontal: true,
              }
            },
      
      xaxis: {
        categories: nom_mic.value,
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
    series_tot_mic= [
        {
          data: tot_mic.value,
        },
    ]
}


onMounted(()=>{
  get_totales_iras()
  get_micro_redes()
  get_iras()
})
</script>

<template>
  <main>
    <div class="container-fluid mb-5">
      <h1 class="tit1 text-center">Seguimiento de IRAS</h1>
      <div class="row">
        <div class="col-md-2">

          <!--iras-->
          <label for="iras" class="col-form-label etiq"><h4 class="tit1">Tipo de IRA</h4></label>
          <select @change="get_tipo_ira(), get_casos_micro_estab_ira(), get_totales_iras_eess(), get_totales_iras_micro(), get_totales_micro()" v-model="cod_ira" id="iras" class="form-select" multiple size="6">
            <!--<option value="" disabled selected>Seleccione una IRA</option>-->
            <option v-for="ira in iras" :value="ira.cod_ira">
              {{ ira.nom_ira }}
            </option>
          </select>
          <!--micro redes-->
          <label for="micro_red" class="col-form-label etiq"><h4 class="tit1">Micro Red</h4></label>
          <select @change="get_nom_mic(), get_eess(), get_casos_micro_estab_ira(), get_totales_iras_eess(), get_totales_iras_micro(), get_totales_micro()" v-model="cod_mic" id="micro_red" class="form-select">
            <option value="" disabled selected>Seleccione una Micro Red</option>
            <option v-for="mic in micro" :value="mic.cod_micro_red">
              {{ mic.nom_micro_red }}
            </option>
          </select>

          <!--establecimientos-->
          <label for="eess" class="col-form-label etiq"><h4 class="tit1">Establecimiento</h4></label>
          <select @change="get_nom_est(), get_casos_micro_estab_ira()" v-model="cod_est" id="eess" class="form-select" multiple size="15">
            <!-- <option value="" disabled selected>Seleccione un establecimiento</option>-->
            <option v-for="est in estabs" :value="est.cod_eess">
              {{ est.nom_eess }}
            </option>
          </select>
          <p style="color:black;visibility: hidden;">
            casos:<p class="datos"> {{ casos }}</p>
            casos micro:{{ casos_tot_micro }}
          </p>


        </div>

        <div class="col-md-10">
          <div class="row gx-5">
            <h2 class="tit1">Totales RED CUSCO NORTE</h2>  
            <div class="col-md-4">
              <div class="card border-primary mb-1 card1">
                <div class="card-header bg-primary text-center text-white"><h5 class="tit1">IRAS</h5></div>
                  <div class="card-body text-primary">
                    <p class="card-text bg-transparent border-primary titulo3 derecha"><h2 class="tit2 text-center">{{tot_ira}}</h2></p>
                  </div>
              </div>
            </div>

            <div class="col-md-4">
              <div class="card border-primary mb-1 card1">
                <div class="card-header bg-primary text-center text-white"><h5 class="tit1">Neumonias</h5></div>
                  <div class="card-body text-primary">
                    <p class="card-text bg-transparent border-primary titulo3 derecha"><h2 class="tit2 text-center ">{{tot_neumonia}}</h2></p>
                  </div>
              </div>
            </div>
            <div class="col-md-4">
              <div class="card border-primary mb-1 card1">
                <div class="card-header bg-primary text-center text-white"><h5 class="tit1">SOB MEN.2A</h5></div>
                  <div class="card-body text-primary">
                    <p class="card-text bg-transparent border-primary titulo3 derecha"><h2 class="tit2 text-center ">{{tot_sob_men_2}}</h2></p>
                  </div>
              </div>
            </div>
            </div>


            <div class="row gx-5 mt-2">
            <div class="col-md-4">
              <div class="card border-primary mb-1 card1">
                <div class="card-header bg-primary text-center text-white"><h5 class="tit1">SOB 2-4A</h5></div>
                  <div class="card-body text-primary">
                    <p class="card-text bg-transparent border-primary titulo3 derecha"><h2 class="tit2 text-center">{{tot_sob_2_4}}</h2></p>
                  </div>
              </div>
            </div>
            <div class="col-md-4">
              <div class="card border-primary mb-1 card1">
                <div class="card-header bg-primary text-center text-white"><h5 class="tit1">Muertes por Neumonia</h5></div>
                  <div class="card-body text-primary">
                    <p class="card-text bg-transparent border-primary titulo3 derecha"><h2 class="tit2 text-center">{{tot_muertes}}</h2></p>
                  </div>
              </div>
            </div>
            <div class="col-md-4">
              <div class="card border-primary mb-1 card1">
                <div class="card-header bg-primary text-center text-white"><h5 class="tit1">Neumonia en Adultos</h5></div>
                  <div class="card-body text-primary">
                    <p class="card-text bg-transparent border-primary titulo3 derecha"><h2 class="tit2 text-center">{{tot_neumonia_adulto}}</h2></p>
                  </div>
              </div>
            </div>
          </div>


          <h2 class="tit1" >Seguimiento de casos por semana/establecimiento</h2>
          <div class="caja p-3 mt-3">
            <h4 class="tit2 text-center text-primary">{{ tipo_ira }} - {{nom_estab}}</h4>
            <VueApexCharts width="100%" height=400px
              :options='options'
              :series='series'
              />
          </div>

          <h2 class="tit1 mt-2">Totales por Establecimiento</h2>  
          <div class="row">
            <div class="col-md-5">
              <div class="caja p-3">
              <DataTable :data="tot" :columns="columns_tot" :options="opciones" class="display hover cell-border table compact table-striped">
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
                :options='options_tot'
                :series='series_tot'
                />
              </div>
            </div>
          </div>

          <h2 class="tit1 mt-2">Seguimiento de casos por semana/Micro Red</h2>  
          <div class="row">
            <div class="col-md-12">
              <div class="caja p-3">
                <VueApexCharts width="100%" height=400px
                :options='options_tot_micro'
                :series='series_tot_micro'
                />
              </div>
            </div>
          </div>

          <div class="row">
            <h1 class="tit1">Totales por Micro Red</h1>
            <div class="col-md-5">
              <div class="caja p-3">
                <DataTable :data="tot_iras_micro" :columns="columns_tot_micro" :options="opciones" class="display hover cell-border table compact table-striped">
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
                :options='options_tot_mic'
                :series='series_tot_mic'
                />
              </div>
            </div>
          </div>


        </div>
      </div>
      

      
    </div>
    
  </main>
</template>


<style scope>
@import 'datatables.net-dt';
.etiq{
  color:#0981F1;
  font-weight: 600;
  font-size: 18px;
}

.tit1{
  font-weight: 800;
}

.tit2{
  font-weight: 600;
  /*color: #0981F1;*/
}
.caja{
  box-shadow: 0px 0px 13px 3px #1e1e1e;
  border-radius: 10px;
}

.derecha{
  text-align: right;
}

.card1{
  box-shadow: 0px 0px 15px 2px #261b5e;
}

.card2{
  box-shadow: 0px 0px 5px 2px #383d3e;
  background-color: rgb(3, 74, 12);
  /*max-width: 350px;*/
  border: #eef0f2;
  height: 110px;
}
</style>
