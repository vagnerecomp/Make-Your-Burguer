<template>
    <div>
        <Message :msg ="msg" v-show="msg" /> <!--passando o valor de msg (do data) para a props msg do componente Message-->
        <div id="form-container">
            <form id="burguer-form" method="POST" @submit="createBurguer">
                <div class="input-container">
                    <label for="nome">Nome do  cliente:</label>
                    <input type="text" id="nome" name="nome" v-model="nome" placeholder="Digite o seu nome">
                </div>
                <div class="input-container">
                    <label for="pao">Escolha o pão:</label>
                    <select name="pao" id="pao" v-model="pao">
                        <option value="">Selecione o seu pão</option>
                        <option v-for="pao in paes" :key="pao.id" :value="pao.tipo">
                            {{pao.tipo}}
                        </option>  <!--Agora temos um v-for que renderiza as options com dados do data() que vieram do backend-->"
                    </select>
                </div>
                <div class="input-container">
                    <label for="carne">Escolha a carne do seu Burguer:</label>
                    <select name="carne" id="carne" v-model="carne">
                        <option value="">Selecione o tipo de carne</option>
                        <option v-for="carne in carnes" :key="carne.id" :value="carne.tipo">
                            {{carne.tipo}}
                        </option> <!--Agora as options vem dinamicamente do data() que rebebe os dados do backend-->"
                    </select>
                </div>
                <div id="opcionais-container" class="input-container">
                    <label id="opcionais-title" for="opcionais">Selecione os opcionais</label>
                    <div class="checkbox-container" v-for="opcional in opcionaisdata" :key="opcional.id">
                        <input type="checkbox" name="opcionais" v-model="opcionais" :value="opcional.tipo">
                        <span>{{opcional.tipo}}</span>
                    </div>
                </div>
                <div class="input-container">
                    <input type="submit" class="submit-btn" value="Criar meu Burguer!">
                </div>
            </form>
        </div>
    </div>
</template>

<script>
import Message from "./Message.vue";
export default {
    name: "BurguerForm",
    data(){
        return{
            // Dados que vem do servidor (todos no plural)
            paes: null,
            carnes: null,
            opcionaisdata: null,
            // Dados que são enviados para o servidor (Todos no singular, com excessão dos opcionais)
            nome: null,
            pao: null,
            carne: null,
            opcionais: [],
            msg: null
        }
    },
    methods: {
        async getIngredientes(){
            const req = await fetch("http://localhost:3000/ingredientes"); //Armazeno o body da rota especificada como json
            const data = await req.json(); //data recebe o body da requisição como um objeto javascript
            // console.log(data) testando para ver se a requisição foi executada

            //Atribuindo os valores obtidos do backend aos dados correspondentes no data() do componente BurguerForm
            //Esses dados serão usados para renderizar as opções de ingradientes na tela com o v-for
            this.paes = data.paes;
            this.carnes = data.carnes;
            this.opcionaisdata = data.opcionais;
        },
        async createBurguer(e){
            e.preventDefault();

            const data= { //Esse objeto representa o hamburguer cadastrado
                nome: this.nome,
                carne: this.carne,
                pao: this.pao,
                opcionais: Array.from(this.opcionais),
                status: "Solicitado"
            }

            const dataJson = JSON.stringify(data); //Convertendo o objeto data para json para enviar ao servidor
           
           //enviando a requisição de POST
            const req = await fetch("http://localhost:3000/burguers", {
                method: "POST",
                headers: { "Content-Type" : "application/json" },
                body: dataJson
            });

            // Posso obter uma resposta do servidor:
            const res = await req.json();
            console.log(res);

            //console.log(data.opcionais) //teste para ver se o método está funcionando

            // colocar Mensagem
            this.msg =`Pedido nº ${res.id} realizado com sucesso`
            // apagar mensagem
            setTimeout(()=> this.msg = "", 3000)
            // apagar os campos
            this.nome = "";
            this.pao = "";
            this.carne = "";
            this.opcionais = [];
        }

    },
    mounted(){
        this.getIngredientes();
    },
    components: {
        Message
    }
}
</script>

<style scoped>
    #burguer-form{
        max-width: 400px;
        margin: 0 auto;
    }
    .input-container{
        display: flex;
        flex-direction: column;
        margin-bottom: 20px;
    }
    label{
        font-weight: bold;
        margin-bottom: 15px;
        color: #222;
        padding: 5px 10px;
        border-left: 4px solid #FCBA03;
    }
    input, select{
        padding: 5px 10px;
        width: 300px;
    }
    #opcionais-container{
        flex-direction: row;
        flex-wrap: wrap;
    }
    #opcionais-title{
        width: 100%;
    }
    .checkbox-container{
        display: flex;
        align-items: flex-start;
        width: 50%;
        margin-bottom: 20px;
    }
    .checkbox-container span,
    .checkbox-container input{
        width: auto; /*Esses elementos vão trabalhar com sua própria largura e não com a largura que eles herdaram das outras regras css
        */
    }
    .checkbox-container span{
        margin-left: 6px;
        font-weight: bold;
    }
    .submit-btn{
        background-color: #222;
        color: #FCBA03;
        font-weight: bold;
        border: 2px solid #222;
        padding: 10px;
        font-size: 16px;
        margin: 0 auto;
        cursor: pointer;    
        transition: 0.5s;
    }
    .submit-btn:hover{
        background-color: transparent;
        color: #222;
    }


</style>