<template>
  <div>
    <div class="title">
      <b-navbar toggleable="lg" type="dark" variant="info">
        <b-navbar-brand class="ml-2">
          <b>{{ appName }}</b>
        </b-navbar-brand>
      </b-navbar>
    </div>
    <div style="border-bottom:1px solid #000; width:max-content; padding:20px;">
      Como é o frete que você precisa?
    </div>
    <div class="d-flex flex-column">
      Destino:
      <select v-model="selected">
        <option value="" disabled selected>Escolha uma conta</option>
        <option v-for="account in accounts" :key="account" :value="account" >{{ account }}</option>
      </select>

      Peso
      <input type="text" v-model="valueKg" placeholder="Insira aqui o peso da carga em KG" />
    </div>

    <button  v-on:click="comverte">Analisar</button>

    <p>Frete mais Barato: {{maisBarato.name}}-R${{maisBarato.cost_transport_heavy}}-{{maisBarato.lead_time}}h</p>

  </div>
</template>

<script>
import {
  BNavbar,
  BNavbarBrand,
} from 'bootstrap-vue'

export default {
  components: {
    BNavbar,
    BNavbarBrand,
  },
  data() {
    const appName = ''

    return {
      appName,
      valueKg:" ",
      accounts: [],
      selected:'',
      objectURL:[],
      maisBarato:[],
      
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
    comverte(){
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
          return {...obj, cost_transport_light: valueCTL[1]*this.valueKg, cost_transport_heavy: valueCTV[1]* this.valueKg, lead_time:ld[0] };
        }

        return obj;
      });

    //trazendo o mais em conta 
      
      if(this.valueKg >= 100){
        const maxHeavy = newArr.reduce(function(prev, current) { 
          return prev.cost_transport_heavy < current.cost_transport_heavy ? prev : current; 
        });
        this.maisBarato=maxHeavy
      console.log(maxHeavy);
      }else if(this.valueKg < 100){
        const maxHeavy = newArr.reduce(function(prev, current) { 
          return prev.cost_transport_heavy < current.cost_transport_heavy ? prev : current; 
        });
        this.maisBarato=maxHeavy
      }



      
    },

    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    // fuctions
    // dictVerification(select ,array){
    //   Object.filter = (obj, predicate) => Object.assign(...Object.keys(obj)
    //                 .filter( key => predicate(obj[key]) )
    //                 .map( key => ({ [key]: obj[key] }) ) );


    //   var filtered = Object.filter(array, score => score > select); 
    //   for (const key in filtered) {
    //     if (Object.hasOwnProperty.call(object, key)) {
    //       const element = object[key];

          
    //     }
    //   }

    // }
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
</style>
