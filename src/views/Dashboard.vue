<template>
  <div>
    <CSelect
      @change="selecionaMes($event)"
      label="Selecione a data"
      :options="mesesCombo"
      :value="mesesCombo"
    />
    <!--
    <ul>
      <li
        class="list-group-item"
        v-for="anoMes of mesesCombo"
        :key="anoMes.id"
        @click="selecionaMes(anoMes)"
      >
        {{ anoMes.id }} - {{ anoMes.label }}
      </li>
    </ul>
-->
    <CCardBody>
      <CButton
        color="primary"
        class="mb-2"
        :href="csvCode"
        download="IPCA.csv"
        target="_blank"
      >
        Download IPCA (.csv)
      </CButton>
      <CDataTable
        :items="posts"
        :fields="fields"
        column-filter
        table-filter
        items-per-page-select
        :items-per-page="5"
        hover
        sorter
        pagination
        @filtered-items-change="setCurrentItems"
      >
        <template #show_details="{ item, index }">
          <td class="py-2">
            <CButton
              color="primary"
              variant="outline"
              square
              size="sm"
              @click="toggleDetails(item, index)"
            >
              {{ Boolean(item._toggled) ? "Hide" : "Show" }}
            </CButton>
          </td>
        </template>
        <template #details="{ item }">
          <CCollapse
            :show="Boolean(item._toggled)"
            :duration="collapseDuration"
          >
            <CCardBody>
              <ul>
                <li
                  class="list-group-item"
                  v-for="(detalh, index) of detalheTabela"
                  :key="detalh.id"
                >
                  <span v-if="index > 0" class="text-muted">
                    {{ detalh.D4N }} - {{ detalh.V }} - {{ detalh.D3N }}
                  </span>
                  <span v-if="index == 0" class="text-muted">
                    Itens medidos
                  </span>
                </li>
              </ul>
            </CCardBody>
          </CCollapse>
        </template>
      </CDataTable>
    </CCardBody>

    <CRow>
      <CCol md="12">
        <CCard>
          <CCardHeader>
            <div>
              <h4>Data início</h4>
            </div>
            <div>
              Data início:
              <CSelect
                label="Selecione a data início"
                :options="mesesCombo"
                :value="mesesCombo"
              />
            </div>
            <div>
              Data fim:
              <CSelect
                label="Selecione a data fim"
                :options="mesesCombo"
                :value="mesesCombo"
              />
            </div>
          </CCardHeader>
          <CCardBody>
            <CRow>
              <CCol md="12">
                <CChartBar
                  :datasets="defaultDatasets"
                  :labels="listaDatasGrafico1"
                />
              </CCol>
            </CRow>
          </CCardBody>
        </CCard>
      </CCol>
    </CRow>

    <CRow>
      <CCol md="12">
        <CCard>
          <CCardHeader>
            <h4>Gráfico B</h4>
          </CCardHeader>
          <CCardBody>
            <CRow>
              <CCol md="12"> gráfico b.... </CCol>
            </CRow>
          </CCardBody>
        </CCard>
      </CCol>
    </CRow>
  </div>
</template>

<script>
import axios from "axios";
import usersData from "./UsersData";

import { CChartBar } from "@coreui/vue-chartjs";

export default {
  name: "DownloadTable",
  components: { CChartBar },
  data() {
    return {
      usersData,
      posts: [],
      currentItems: usersData.slice(),
      mesesCombo: [],
      listaTratada: [],
      anoMesInicial: {},
      result: {},
      fields: [],
      details: [],
      collapseDuration: 0,
      cabecalhoLista: {},
      detalheTabela: {},
      listaPura: [],
      listaItemPaiGrafico1: [],
      listaDatasGrafico1: [],
    };
  },
  created() {
    this.criaCabecalho();
    axios
      .get(`https://api-psi-geadv.herokuapp.com/`)
      .then((response) => {
        this.posts = response.data;
        this.cabecalhoLista = this.posts.shift();
        this.ordernaPorData(this.posts);
        this.trataListaItemPaiGrafico1();
      })
      .catch((e) => {
        //this.errors.push(e);
      });
  },
  methods: {
    setCurrentItems(val) {
      this.currentItems = val;
    },
    groupByAnoMesCombo(lista) {
      var listaFinalCombo = [];
      var id = this.groupByAnoMes(lista, "D3C");
      var descricao = this.groupByAnoMes(lista, "D3N");
      var i = 0;
      id.forEach(function (dado) {
        var registro = {};
        registro.id = dado;
        registro.label = descricao[i];
        listaFinalCombo.push(registro);
        i++;
      });
      this.anoMesInicial = listaFinalCombo.reverse()[0];
      this.posts = this.result[this.anoMesInicial.id];
      this.tratamentoFinalTabela(this.posts);
      return listaFinalCombo;
    },
    ordernaPorData(lista) {
      this.result = lista.reduce(function (r, a) {
        r[a.D3C] = r[a.D3C] || [];
        r[a.D3C].push(a);
        return r;
      }, Object.create(null));
      this.mesesCombo = this.groupByAnoMesCombo(lista);
    },
    groupByAnoMes(objectArray, property) {
      var anoMes = [];
      var i = 0;
      objectArray.reduce(function (acc, obj) {
        if (i > 0) {
          var key = obj[property];
          anoMes.push(key);
          return acc;
        }
        i++;
      }, {});
      var unico = anoMes.filter(function (elem, index, self) {
        return index === self.indexOf(elem);
      });
      return unico;
    },
    selecionaMes(anoMes) {
      this.anoMesInicial = this.mesesCombo[anoMes.target.selectedIndex];
      this.tratamentoFinalTabela(this.result[this.anoMesInicial.id]);
    },
    carregaDadosGrafico1(mesInicio, mesFim) {
      var listaGrafico = [];
      var conta = 0;
      var arrayMeses = [];

      var result = this.posts.reduce(function (r, a) {
        r[a.D3C] = r[a.D3C] || [];
        r[a.D3C].push(a);
        return r;
      }, Object.create(null));

      for (var i = mesInicio; i <= mesFim; i++) {
        arrayMeses.push(i);
        result;
      }

      var listabackgroundColor = [];
      listabackgroundColor.push("rgb(228,102,81,0.9)");
      listabackgroundColor.push("rgb(255, 145, 0)");
      listabackgroundColor.push("rgb(136, 0, 246)");
      listabackgroundColor.push("rgb(205, 25, 108)");

      var listaGraficoData = [];
      listaGraficoData.push([30, 39, 25, 55, 30, 33, 35]);
      listaGraficoData.push([28, 43, 35, 22, 36, 53, 35]);
      listaGraficoData.push([33, 23, 25, 50, 54, 60, 27]);
      listaGraficoData.push([35, 54, 22, 45, 32, 53, 35]);

      this.listaItemPaiGrafico1.forEach(function (titulo) {
        var objetoGrafico = {};
        objetoGrafico.label = titulo.label;
        objetoGrafico.backgroundColor = listabackgroundColor[conta];
        objetoGrafico.data = listaGraficoData[conta];
        listaGrafico.push(objetoGrafico);
        conta++;
      });

      this.listaDatasGrafico1 = arrayMeses;

      return listaGrafico;
    },
    tratamentoFinalTabela(lista) {
      var itemAtual = null;
      var listaFinal = [];
      var contador = 0;
      this.listaPura = this.result[this.anoMesInicial.id];
      for (var i = 0; i < lista.length; i++) {
        if (lista[i].D2N != itemAtual) {
          itemAtual = lista[i].D2N;
          var objFinal = {};
          // objFinal[this.fields[0].key] = lista[i].NC;
          //   objFinal[this.fields[1].key] = lista[i].NN;
          //  objFinal[this.fields[2].key] = lista[i].D1C;
          //     objFinal[this.fields[0].key] = lista[i].D1N;
          //     objFinal[this.fields[1].key] = lista[i].D2C;
          objFinal[this.fields[0].key] = lista[i].D2N;
          objFinal[this.fields[1].key] = lista[i].D3C;
          objFinal[this.fields[2].key] = lista[i].D3N;
          objFinal[this.fields[3].key] = lista[i].D4C;
          objFinal[this.fields[4].key] = lista[i].D4N;
          objFinal[this.fields[5].key] = lista[i].MC;
          objFinal[this.fields[6].key] = lista[i].MN;
          objFinal[this.fields[7].key] = lista[i].V;
          objFinal.id = contador;
          objFinal._toggled = null;
          var itemGrafico = {};
          itemGrafico.label = lista[i].D2N;
          itemGrafico.backgroundColor = "";
          this.listaItemPaiGrafico1.push(itemGrafico);
          listaFinal.push(objFinal);
          contador++;
        }
      }

      this.posts = listaFinal;
    },
    toggleDetails(item) {
      this.$set(this.posts[item.id], "_toggled", !item._toggled);
      this.collapseDuration = 300;
      this.$nextTick(() => {
        this.collapseDuration = 0;
      });
      this.detalheTabela = this.carregaDetalhe(item);
    },
    criaCabecalho() {
      this.fields = [
        //   { key: "Nível Territorial (Código)", _style: "min-width:100px;" },
        //   { key: "Nível Territorial", _style: "min-width:100px;" },
        //    { key: "Brasil (Código)", _style: "min-width:100px;" },
        //    { key: "Brasil", _style: "min-width:100px;" },
        //    { key: "Variável (Código)", _style: "min-width:100px;" },
        { key: "Variável", _style: "min-width:100px;" },
        { key: "Mês (Código)", _style: "min-width:100px;" },
        { key: "Mês", _style: "min-width:150px;" },
        {
          key: "Geral, grupo, subgrupo, item e subitem (Código)",
          _style: "min-width:100px;",
        },
        {
          key: "Geral, grupo, subgrupo, item e subitem",
          _style: "min-width:100px;",
        },
        { key: "Unidade de Medida (Código)", _style: "min-width:100px;" },
        { key: "Unidade de Medida", _style: "min-width:100px;" },
        { key: "Valor", _style: "min-width:100px;" },
        {
          key: "show_details",
          label: "",
          _style: "width:1%",
          sorter: false,
          filter: false,
        },
      ];
    },
    carregaDetalhe(item) {
      var listaDetalhe = this.listaPura.reduce(function (r, a) {
        r[a.D2N] = r[a.D2N] || [];
        r[a.D2N].push(a);
        return r;
      }, Object.create(null));
      return listaDetalhe[item["Variável"]];
    },
  },
  computed: {
    csvContent() {
      return this.currentItems
        .map((item) => Object.values(item).join(","))
        .join("\n");
    },
    csvCode() {
      return (
        "data:text/csv;charset=utf-8,SEP=,%0A" +
        encodeURIComponent(this.csvContent)
      );
    },
    defaultDatasets() {
      return this.carregaDadosGrafico1("201901", "201907");
    },
  },
};
</script>