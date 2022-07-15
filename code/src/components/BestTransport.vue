<template>
  <div>
    <div class="title">
      <b-navbar toggleable="lg" type="dark" variant="info">
        <b-navbar-brand class="ml-2">
        <img src="../assets/logo.png" width="40px" height="40px" style="margin-right: 5px; color: #fff;" >
          <b>{{ appName }}</b>
        </b-navbar-brand>
      </b-navbar>
    </div>
    <div  class="containerGeral">
      <div class="text">
        <p>Como é o frete que você precisa?</p>
      </div>
        <div class="containerInput">
          Destino:
          <select v-model="selected">
            <option value="" disabled selected>Selecione aqui o destino do frete.</option>
            <option v-for="account in accounts" :key="account" :value="account" >{{ account }}</option>
          </select>

          Peso
          <input type="text" v-model="valueKg" placeholder="Insira aqui o peso da carga em KG" />

          <div class="d-flex w-100 justify-content-center">
            <button  v-on:click="result">Analisar</button>
          </div>
        </div>

        

        <div v-if="show">
          <div class="text">
            <p>Estas são as melhores alternativas de frete que encontramos para você.</p>
          </div>
          <div class="result" style="background-color: #d9ead3;">
              <p>
                <img src="../assets/iconsMao.png" alt="icon_mao" width="40px" height="40px">
                Frete mais Barato: <b> {{maisBarato}}</b></p>
          </div>
          <div class="result" style="background-color: #c9daf8;">
                <p>
                  <img src="../assets/watch.png" alt="icon_mao" width="40px" height="40px">
                  Frete mais Rápido: <b>{{maisRapido}}</b></p>
          </div>
        </div>
    </div>
  </div>
</template>

<script>
import {
  BNavbar,
  BNavbarBrand,
} from 'bootstrap-vue';
export default {
  components: {
    BNavbar,
    BNavbarBrand,
  },
  data() {
    const appName = ''

    return {
      appName,
      valueKg:"",
      accounts: [],
      selected:'',
      objectURL:[],
      maisBarato:"",
      maisRapido:"",
      show:false,
    }
  },
  created() {
    // Implemente aqui o GET dos dados da API REST
    // para que isso ocorra na inicialização da pagina
    let url="http://localhost:3000/transport";
      var list=[];
      fetch(url).then(res=>{return res.json()})
                .then(json=>{
                  for (const key in json) {
                    if (Object.hasOwnProperty.call(json, key)) {
                      const objectElment = json[key];
                      this.objectURL.push(objectElment)
                      const element = json[key].city;
                      list.push(element)
                    }
                  }
                  list.sort();
                  const  valueFinalList= list.filter(function(ele , pos){
                    return list.indexOf(ele) == pos;
                  })
                  for (const key in valueFinalList) {
                    if (Object.hasOwnProperty.call(valueFinalList, key)) {
                      const element = valueFinalList[key];
                      this.accounts.push(element);
                    }
                  } 
                })
    this.appName = 'Melhor Frete'
  },
  methods: {
    // Implemente aqui os metodos utilizados na pagina
    verification(){
      const array=[]
      for (const key in this.objectURL) {
        if (Object.hasOwnProperty.call(this.objectURL, key)) {
          const element = this.objectURL[key];
          if(element.city == this.selected){
            array.push(element)
          }
        }
      } 
      console.log(array)    
      
      // mudando valores

      const newArr = array.map(obj => {
        if (obj.city === this.selected) {
          let valueCTL=obj.cost_transport_light.split(" ")
          let valueCTV=obj.cost_transport_heavy.split(" ")
          let ld=obj.lead_time.split("h")
          return {...obj, cost_transport_light: (parseFloat(valueCTL[1])*this.valueKg).toFixed(1), cost_transport_heavy: (parseFloat(valueCTV[1])* this.valueKg).toFixed(1), lead_time:parseInt(ld[0]) };
        }

        return obj;
      });
      console.log(newArr)

      
        
      if(this.valueKg > 100){
        console.log('1')
          // trazendo opção mais barata
          const maxHeavy = newArr.reduce(function(prev, current) { 
            return prev.cost_transport_heavy < current.cost_transport_heavy ? prev : current; 
          });
          this.maisBarato=maxHeavy.name+"-R$"+maxHeavy.cost_transport_heavy+"-"+maxHeavy.lead_time+"h"
          console.log(maxHeavy)

          // trazendo opção mais rapida
          const moreLead_time = newArr.reduce(function(prev, current) { 
            return prev.lead_time < current.lead_time ? prev : current; 
          });
          this.maisRapido=moreLead_time.name+"-R$"+moreLead_time.cost_transport_heavy+"-"+moreLead_time.lead_time+"h"

          console.log(moreLead_time)


      }else if(this.valueKg <= 100){
        console.log('2')
          // trazendo valor mais barato
          const maxLigt = newArr.reduce(function(prev, current) { 
            return prev.cost_transport_light < current.cost_transport_light ? prev : current; 
          });
          this.maisBarato=maxLigt.name+"-R$"+maxLigt.cost_transport_light+"-"+maxLigt.lead_time+"h"
          console.log(maxLigt)

          // trazendo opção mais rapida
          const moreLead_time = newArr.reduce(function(prev, current) { 
            return prev.lead_time < current.lead_time ? prev : current; 
          });
          console.log(newArr)
          console.log(moreLead_time)
          this.maisRapido=moreLead_time.name+"-R$"+moreLead_time.cost_transport_light+"-"+moreLead_time.lead_time+"h"

      }

      this.show = true;
    },

    result(){
      if(this.selected === "" || this.valueKg === ""){
        alert("Por favor preencha todos os campos para podemos fazer a análise.")
      }
      else if(isNaN(this.valueKg) != false){
        alert("Por favor insira um numero. Ex.: 123")
      }
      else{
        this.verification();
      }

    }
  },
}
</script>

<style scoped>
.title .navbar {
  background-color: #00aca6 !important;
}

.title .navbar-brand {
  margin-left: 20px;
}
.containerGeral{
  display: flex;
  flex-direction: column;
  margin-left: 10%;
}
select, input{
  height: 40px;
  margin-bottom: 10px;
}

.containerInput{
  width: 400px;
  display: flex;
  flex-direction: column;
  margin-left: 20px;
}

.text{
  margin-top: 20px;
  border-bottom:1px solid #000; 
  width:450px;
  margin-bottom: 30px;
}

p{
  font-size: 20px;
  font-weight: bold;
}

.result p{
  font-weight: normal;
  font-size: 17px;
}

.result{
  width:450px;
  padding: 5px;
  border-radius: 5px;
  margin: 10px;
  margin-left: 0;
  padding-top: 20px;
  align-items: center;
  border-style: dashed;
}
button{
  background-color: #00aca6;
  border: 1px solid;
  color: #000;
  font-weight: bold;
  width: 100px;
  border-radius: 5px;
}
</style>
