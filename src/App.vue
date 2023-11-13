<script setup>
import { RouterLink, RouterView } from 'vue-router'
import Menu from  './components/icons/Menu.vue'
import Close from './components/icons/Close.vue'
import Home from './components/icons/Home.vue'
import Pulmon from './components/icons/Pulmon.vue'
import Estomago from './components/icons/Estomago.vue'
</script>

<template>
  <input type="checkbox" id="check">
  <label for="check">
    <Menu id="btn"/>
    <Close id="cancel"/>
  </label>
  <div class="sidebar">
    <header>Menu</header>
    <ul>
      <li class="active">
         <RouterLink class="active" to="/"><Pulmon class="icono"/>Monitoreo IRAS</RouterLink>
      </li>
      <li><RouterLink to="/edas"><Estomago class="icono"/>Monitoreo EDAS</RouterLink></li>
    </ul>
    
  </div>  
    <RouterView />
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@100;200;400;600;700;800;900&display=swap');

*{
  margin: 0;
  padding: 0;
  text-decoration: none;
}

body{
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100vw;
  height: 100vh;
  font-family: 'Poppins';
  font-weight: 400;
}

.sidebar{
  top:0;
  position: fixed;
  width: 240px;
  left: -240px;
  height: 100%;
  box-shadow: 0px 0px 8px 3px #4c4a4a;
  background-color: #0558ac;
  transition: all .5s ease;
}
.sidebar header{
  font-size: 28px;
  font-weight: 400;
  color: #fefefe;
  line-height: 60px;
  text-align: center;
  background-color: #0558ac;
}

.sidebar ul li a{
  display: flex;
  height: 60px;
  width: 100%;
  color: #ffffff;
  line-height: 60px;
  align-items: center;
  border-radius: 10px;
  box-sizing: border-box;
  transition: 0.5s;
  transition: all 100ms ease;
  text-transform: uppercase;
}
.sidebar li{
  display: block;
  height: 60px;
  width: 100%;
  color: #ffffff;
  margin-top: 3px;
  line-height: 60px;
  box-sizing: border-box;
  font-family: 'Lato', sans-serif;
  transition: all .5s ease;
}

.sidebar > ul > li > a.active,a:hover{
  width: 100%;
  color: #fff;
  background: #10b4ef;
  border-radius: 5px;
  box-shadow: 0px 0px 5px 2px #444343;;
}

#check{
  display: none;
}

label #btn, label #cancel{
  position: fixed;
  left: 0;
  top:0;
  cursor: pointer;
  color: #82daf0;
  border-radius: 5px;
  margin: 5px 5px;
  font-size: 24px;
  background-color: #3a26f0;
  box-shadow:inset 2px 2px 2px 0px rgba(255,255,255,.5),
    inset -7px -7px 10px 0px rgba(0,0,0,.1),
   3.5px 3.5px 20px 0px rgba(0,0,0,.1),
   2px 2px 5px 0px rgba(0,0,0,.1);
  height: 65px;
  width: 65px;
  text-align: center;
  text-shadow: 2px 2px 3px rgba(255,255,255,0.5);
  line-height: 65px;
  transition: all .5s ease;
}
label #cancel{
  opacity: 0;
  visibility: hidden;
}
#check:checked ~ .sidebar{
  left: 0;
}
#check:checked ~ label #btn{
  margin-left: 245px;
  opacity: 0;
  visibility: hidden;
}
#check:checked ~ label #cancel{
  margin-left: 245px;
  opacity: 1;
  visibility: visible;
}
@media(max-width : 860px){
  .sidebar{
    height: auto;
    width: 70px;
    left: 0;
    margin: 100px 0;
  }
  header,#btn,#cancel{
    display: none;
  }
  .sidebar ul li{
    position: absolute;
    margin-left: 23px;
    opacity: 0;
    visibility: hidden;
  }
  .sidebar ul li a{
    height: 60px;
  }
  .icono{
    margin-left: -10px;
  }
  a:hover {
    width: 200px;
    background: inherit;
  }
  .sidebar ul li a:hover{
    opacity: 1;
    visibility: visible;
  }
}

h2 {
  position: relative;
  text-align: center;
  color: #353535;
  font-size: 60px;
  font-family: 'Lato', sans-serif;
  margin: 0;
  color: #a759f5;
}

p {
  font-family: 'Lato', sans-serif;
  font-weight: 300;
  text-align: center;
  font-size: 30px;
  color: #d6adff;
  margin: 0;
}

.icono{

  height: 60px;
  text-align: center;
  line-height: 60px;
  margin-right: 10px;
  text-align: center;
  width: 40px;
  height: 40px;
}


</style>
