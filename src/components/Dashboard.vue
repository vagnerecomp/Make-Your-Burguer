<template>
    <div id="burguer-table">
        <Message :msg ="msg" v-show="msg" />
        <div>
            <div id="burguer-table-heading">
                <div class="order-id">#:</div>
                <div>Cliente:</div>
                <div>Pão:</div>
                <div>Carne:</div>
                <div>Opcionais:</div>
                <div>Ações:</div>
            </div>
            <div id="burguer-table-rows">
                <div class="burguer-table-row" v-for="burguer in burguers" :key="burguer.id">
                    <div class="order-number"> {{ burguer.id }}</div>
                    <div>{{ burguer.nome }}</div>
                    <div>{{ burguer.pao }}</div>
                    <div>{{ burguer.carne }}</div>
                    <div>
                        <ul>
                            <li v-for="(opcional, index) in burguer.opcionais" :key="index">
                                {{ opcional }}
                            </li>
                        </ul>
                    </div>
                    <div>
                        <select name="status" class="status" @change="updateBurguer($event, burguer.id)">
                            <option value="">Selecione</option>
                            <option v-for="state in status" :key="state.id" :selected="burguer.status == state.tipo" :value="state.tipo">
                                {{ state.tipo }}
                            </option>
                        </select>
                        <button class="delete-btn" @click="deleteBurguer(burguer.id)"> <!--Eu tenho acesso ao burguer porque, como esse butão é um elemento filho de .burguer-table-row, ainda estamos no loop v-for="burguer in burguers"-->
                            Cancelar
                        </button>
                    </div>
                </div>
                
            </div>
        </div>
    </div>
</template>

<script>
import Message from "./Message.vue";
export default {
    name: 'Dashboard',
    data(){
        return{
            burguers: null,
            burguer_id: null,
            status: [],
            msg: null
        }
    },
    components: {
        Message
    },
    methods: {
        async getPedidos() {
            const req = await fetch('http://localhost:3000/burguers');
            const data = await req.json();
            this.burguers = data;
            console.log(this.burguers);
            // Resgata os status de pedidos
            this.getStatus();
      },
      async getStatus(){
          const req = await fetch('http://localhost:3000/status');
          const data = await req.json();
          this.status = data;
          //   console.log(this.status);
      },
      async deleteBurguer(id){ //Fiz a requisição interpolando o parâmetro id
          const req = await fetch(`http://localhost:3000/burguers/${id}`, {
              method: "DELETE"
          });
          
          const res = await req.json();
          this.getPedidos(); //Forçando uma atualização do sistema por meio do backend
          //Eu poderia fazer essa atualização manualmente, deletando os itens correspondentes no burguers do data()
          //Mas cabe a mim analisar qual implementação se adequa mais a situação
          // No caso dessa implementação, chamando o this.getPedidos() no deleteBurguer(), o sistema faz 3 requisições: GET Pedidos/ GET Status/ DELETE burguer
          //Até aqui fizemos Create, Read e Delete... Falta apenas o Update para completar um sistema de CRUD.   

          this.msg =`Pedido nº ${id} cancelado com sucesso`
          // apagar mensagem
          setTimeout(()=> this.msg = "", 3000)
      },
      async updateBurguer(event, id){
          const option = event.target.value;
          const dataJson = JSON.stringify({status: option});
          const req = await fetch(`http://localhost:3000/burguers/${id}`, {
              method: "PATCH", //É como se fosse o UPDATE mas só atualiza o que estamos enviando no bodyx
              headers: {"Content-Type" : "application/json"},
              body: dataJson
          });

          const res = await req.json();
          console.log(res);

          this.msg =`Pedido nº ${res.id} foi atualizado para ${res.status}`
            // apagar mensagem
            setTimeout(()=> this.msg = "", 3000)
      }

      
    
    },
    mounted(){
        this.getPedidos(); 
    }
}
</script>

<style scoped>
    #burguer-table{
        max-width: 1200px;
        margin: 0 auto; 
    }
    #burguer-table-heading,
    #burguer-table-rows,
    .burguer-table-row{
        display: flex;
        flex-wrap: wrap; /*colocando um item ao lado do outro nan horizontal*/
    }
    #burguer-table-heading{
        font-weight: bold;
        padding: 12px;
        border-bottom: 3px solid #333;
    }
    #burguer-table-heading div,
    .burguer-table-row div{
        width: 19%;
    }
    .burguer-table-row{
        width: 100%;
        padding: 12px;
        border-bottom: 1px solid #ccc;
    }
    #burguer-table-heading .order-id,
    .burguer-table-row .order-number{
        width: 5%;
    }
    select{
        padding: 12px 6px;
        margin-right: 12px;
    }
    .delete-btn{
        background-color: #222;
        color: #FCBA03;
        font-weight: bold;
        border: 2px solid #222;
        padding: 10px;
        font-size: 16;
        margin: 0 auto;
        cursor: pointer;
        transition: 0.5s;
    }
    .delete-btn:hover{
        background-color: transparent;
        color: #222;
    }
    
</style>