<template>
    <div class="modal-overlay">
        <div v-if="this.isAuthenticated" class="etapas">
            <div v-if="etapaAtual === 1" class="etapa">
                <div class="cabecalho">
                    <p class="titulo-etapa">Agendar Aula</p>
                    <div class="informacoes">
                        <p class="nome-professor">{{this.currentTutor['nome']}}</p>
                        <p class="info-adicional">Sessões de 1 hora aula (50 min)</p>
                    </div>
                </div>
                <div class="selecao-dias">
                    <div v-for="dia in diasSemana" :key="dia" class="dia" :class="{ 'selecionado': diaSelecionado === dia }" @click="selecionarDia(dia)">
                        {{ dia }}
                    </div>
                </div>
                <div class="horarios">
                    <div v-for="horario in horarios[this.diaSelecionado]" :key="horario" class="horario" :class="{ 'selecionado': horaSelecionada === horario }" @click="selecionarHora(horario)">
                        {{ horario["hora"] }}
                    </div>
                </div>
                <button @click="redirectToProfessor">Cancelar</button>
                <button @click="proximo" class="botao-proxima-etapa">Próximo</button>
            </div>

            <div v-if="etapaAtual === 2" class="etapa">
                <div class="cabecalho">
                    <p class="titulo-etapa">Agendar Aula</p>
                    <div class="informacoes">
                        <p class="nome-professor">{{this.currentTutor['nome']}}</p>
                        <p class="info-adicional">Sessões de 1 hora aula (50 min)</p>
                    </div>
                    <p>Qual o assunto que deseja estudar?</p>
                    <p style="color: darkgrey">Selecione todas as opções desejáveis</p>

                </div>
                <div class="assunto-selecao">
                    <p>Selecione todas as opções desejáveis</p>
                    <input type="text" style="width: 95%;" placeholder="Pesquise por assunto" v-model="pesquisaAssunto" />
                    <div v-for="(area, index) in areas" :key="index">
                        <p>{{ area.nome }}</p>
                        <div class="linha-sutil"></div>
                        <div v-for="(assunto, aIndex) in area.assuntos" :key="aIndex" class="assunto">
                            <input type="checkbox" v-model="assuntosSelecionados" :value="assunto" />
                            <label>{{ assunto }}</label>
                        </div>
                    </div>
                </div>
                <button @click="etapaAtual--">Voltar</button>
                <button @click="proximo" class="botao-proxima-etapa">Próximo</button>
            </div>

            <div v-if="etapaAtual === 3" class="etapa">
                <div class="cabecalho">
                    <p class="titulo-etapa">Agendar Aula</p>
                    <div class="informacoes">
                        <p class="nome-professor">{{this.currentTutor['nome']}}</p>
                        <p class="info-adicional">Sessões de 1 hora aula (50 min)</p>
                    </div>
                </div>
                <div class="avaliacao-assuntos">
                    <div v-for="(assunto, index) in assuntosSelecionados" :key="index">
                        <p>{{ assunto }}</p>
                        <div class="nota-assunto">
                            <span>Nota: </span>
                            <input type="number" v-model="notasAssuntos[index]" min="1" max="7" />
                        </div>
                    </div>
                </div>
                <button @click="etapaAtual--">Voltar</button>
                <button @click="proximo" class="botao-proxima-etapa">Próximo</button>
            </div>

            <div v-if="etapaAtual === 4" class="etapa">
                <div class="cabecalho">
                    <p class="titulo-etapa">Agendar Aula</p>
                    <div class="informacoes">
                        <p class="nome-professor">{{this.currentTutor['nome']}}</p>
                        <p class="info-adicional">Sessões de 1 hora aula (50 min)</p>
                    </div>
                </div>
                <div class="maiores-dificuldades">
                    <textarea style="width: 95%;" placeholder="Descreva suas maiores dificuldades nesse(s) assunto(s)" v-model="maioresDificuldades"></textarea>
                </div>
                <button @click="etapaAtual--">Voltar</button>
                <button @click="proximo" class="botao-proxima-etapa">Próximo</button>
            </div>

            <div v-if="etapaAtual === 5" class="etapa">
                <div class="cabecalho">
                    <p class="titulo-etapa">Agendar Aula - Confirmação</p>
                    <div class="informacoes">
                        <p class="nome-professor">{{this.currentTutor['nome']}}</p>
                        <p class="info-adicional">Sessões de 1 hora aula (50 min)</p>
                    </div>
                </div>
                <div class="resumo-dados">
                    <p><span>Assuntos Selecionados:</span> {{ assuntosSelecionados.join(', ') }}</p>
                    <p><span>Notas dos Assuntos:</span> {{ notasAssuntos.join(', ') }}</p>
                    <p><span>Maiores Dificuldades:</span> {{ maioresDificuldades }}</p>
                </div>
                <button @click="etapaAtual--">Voltar</button>
                <button @click="confirmarAgendamento" class="botao-proxima-etapa">Confirmar Agendamento</button>
            </div>
        </div>
    </div>
</template>

<script>
import { mapGetters } from 'vuex';
export default {
    props: ['viewCardFunc', 'tutor'],
    name: 'Etapas',
    computed: {
        ...mapGetters(['isAuthenticated', 'user']),
    },
    data() {
        return {
            usuarioAutenticado: false,
            etapaAtual: 1,
            diasSemana: [],
            diaSelecionado: null,
            horaSelecionada: null,
            currentTutor: {},
            horarios: {
                "Seg": [],
                "Ter": [],
                "Qua": [],
                "Qui": [],
                "Sex": [],
                "Sáb": [],
                "Dom": []
            },
            pesquisaAssunto: '',
            areas: [],
            assuntosSelecionados: [],
            notasAssuntos: [],
            maioresDificuldades: '',

        };
    },
    async mounted(){
        this.currentTutor = JSON.parse(JSON.stringify(this.tutor))

        let result = await this.$store.dispatch('getHorarios');
        if (result) {
            for (let i = 0; i < result.data.length; i++){
                if (result.data[i]['usuario_id'] == this.currentTutor['id']) {
                    const data = new Date(result.data[i]['data']);
                    let diaDaSemana = data.toLocaleDateString('pt-BR', { weekday: 'long' });
                    diaDaSemana = diaDaSemana.charAt(0).toUpperCase() + diaDaSemana.slice(1)
                    diaDaSemana = diaDaSemana.slice(0,3)

                    if (!this.diasSemana.includes(diaDaSemana)) {
                        this.diasSemana.push(diaDaSemana)
                    }

                    if (this.diaSelecionado === null)
                        this.diaSelecionado = diaDaSemana

                    this.horarios[diaDaSemana].push({"hora": result.data[i]['hora'], "id": result.data[i]['id']})
                }
            }
        } else {
            alert('Falha no login. Verifique suas credenciais.');
        }

        this.getAreas()
    },
    methods: {
        redirectToProfessor() {
            this.viewCardFunc()
        },
        proximo() {
            // Avança para a próxima etapa
            this.etapaAtual++;
        },
        selecionarDia(dia) {
            this.diaSelecionado = dia;
        },
        async confirmarAgendamento() {
            const user = JSON.parse(JSON.stringify(this.user))

            let result = await this.$store.dispatch('getStatus');
            let status_type = undefined
            for (let i = 0; i < result.data.length; i++){
                if (result.data[i]['descricao'] === 'EM ANALISE')
                {
                    status_type = result.data[i]['id']
                }
            }

            const assuntos = JSON.parse(JSON.stringify(this.assuntosSelecionados));
            const notas = JSON.parse(JSON.stringify(this.notasAssuntos));

            let notas_assuntos = []
            for (let i = 0; i < assuntos.length; i++){
                notas_assuntos.push({
                    "nota": notas[i],
                    "assunto": assuntos[i]
                })
            }

            const send = {
                horario_id: JSON.parse(JSON.stringify(this.horaSelecionada))['id'],
                aluno_id: user.data.id,
                tutor_id: this.currentTutor.id,
                notas_Assuntos: notas_assuntos,
                maiores_dificuldades: JSON.parse(JSON.stringify(this.maioresDificuldades)),
                tutoria_status_id: status_type,
                link: "vazio",
            }

            //console.log(send);

            let schedule = await this.$store.dispatch('scheduleTutoring', send);
            console.log(schedule)
            if (schedule) {
                this.$router.push('/')
            } else {
                alert('Falha ao marcar a tutoria');
            }
        },
        getHorarios() {

        },
        selecionarHora(hora){
            this.horaSelecionada = hora
        },
        async getAreas(){
            let result = await this.$store.dispatch('getSpeciality');
            let specialityNames = await this.$store.dispatch('getSpecialityNames');

            const areas = [
                {
                    nome: 'Álgebra',
                    assuntos: ['Equações do 1º grau', 'Equações do 2º grau', 'Sistema de Equações', 'Funções e Gráficos']
                },
                {
                    nome: 'Geometria',
                    assuntos: ['Geometria Plana e espacial', 'Geometria Analítica', 'Teorema de Pitágoras']
                },
                {
                    nome: 'Estatística',
                    assuntos: ['Estatística Descritiva', 'Probabilidade', 'Inferência Estatística', 'Análise de Regressão', 'Estatística Multivariada']
                },
                {
                    nome: 'Matrizes e Determinantes',
                    assuntos: ['Operações com Matrizes', 'Determinantes e Regra de Cramer', 'Matrizes Inversas']
                },
                {
                    nome: 'Trigonometria',
                    assuntos: ['Funções Trigonométricas', 'Identidades Trigonométricas', 'Trigonometria Esférica']
                }
            ]

            let speciality_ids = []
            let subjects_temp = []
            if (result) {
                for(let i = 0; i < result.data.length; i++)
                {
                    if (result.data[i]['usuario_id'] == this.currentTutor['id']) {
                        speciality_ids.push(result.data[i]['especialidade_id'])
                    }
                }

                for(let i = 0; i < speciality_ids.length; i++)
                {
                    for(let j = 0; j < specialityNames.data.length; j++)
                    {
                        if(specialityNames.data[j]['id'] == speciality_ids[i]){
                            subjects_temp.push(specialityNames.data[j]['descricao'].toLowerCase())
                        }
                    }
                }


                for(let i = 0; i < areas.length; i++)
                {
                    let name = areas[i]['nome'].toLowerCase();

                    if (subjects_temp.includes(name)) {
                        this.areas.push(areas[i])
                    }
                }

            } else {
                alert('Falha na solitação das especialidades.');
            }
        }
    }
};
</script>

<style scoped>
.resumo-dados {
    margin-top: 20px;
}

.resumo-dados p {
    margin-bottom: 10px;
}

.modal-overlay{
    position: fixed;
    width: 100%;
    height: auto;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background-color: #0000007f;
    z-index: 1;
    display: flex;
    justify-content: center;
    align-items: center;
}

.resumo-dados p span {
    font-weight: bold;
}

.maiores-dificuldades textarea {
    width: 100%;
    height: 100px;
    padding: 10px;
}

.avaliacao-assuntos {
    margin-top: 20px;
}

.avaliacao-assuntos p {
    font-weight: bold;
}

.nota-assunto {
    display: flex;
    align-items: center;
    margin-bottom: 10px;
}

.nota-assunto span {
    margin-right: 10px;
}

.nota-assunto input {
    width: 40px;
    padding: 5px;
}

.assunto-selecao {
    margin-top: 20px;
}

.assunto-selecao p {
    color: grey;
}

.assunto-selecao input[type="text"] {
    width: 100%;
    padding: 10px;
    margin-bottom: 10px;
}

.assunto {
    display: flex;
    align-items: center;
    margin-bottom: 5px;
}

.assunto label {
    margin-left: 10px;
}

.linha-sutil {
    border-top: 1px solid #ccc;
    margin: 5px 0;
}

.etapas {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: white;
    width: 35%;
    max-width: 800px;
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.cabecalho {
    display: flex;
    align-items: center;
    margin-bottom: 10px;
}

.titulo-etapa {
    color: #003773;;
    font-size: 1.2em;
    font-weight: bold;
    margin-right: 10px;
}

.nome-professor {
    font-weight: bold;
}

.info-adicional {
    font-size: 0.8em;
    color: grey;
    margin-left: 10px;
}

.selecao-dias {
    display: flex;
    margin-bottom: 10px;
}

.dia {
    flex: 1;
    padding: 10px;
    text-align: center;
    cursor: pointer;
    border: 1px solid #ccc;
    border-radius: 5px;
    margin-right: 5px;
}

.dia.selecionado {
    background-color: #003773;;
    color: white;
}

.horario.selecionado {
    background-color: #003773;;
    color: white;
}

.horarios {
    display: flex;
    flex-direction: column;
}

.horario {
    padding: 10px;
    margin-bottom: 5px;
    border: 1px solid #ccc;
    border-radius: 5px;
    text-align: center;
}

.botao-proxima-etapa {
    background-color: #003773;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 5px;
    cursor: pointer;
    float: right;
}
</style>
