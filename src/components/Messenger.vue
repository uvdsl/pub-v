<template>
  <div class="p-grid">
    <div class="p-inputgroup p-col-6 p-row p-offset-3">
      <Button icon="pi pi-search" @click="search" />
      <InputText
        placeholder="Name of Person"
        v-model="name"
        @keyup.enter="search"
      />
      <Button label="Search" @click="search" />
    </div>
      <div v-if="search_result.length !== 0" class="p-text-center p-col-6 p-row p-offset-3">
    <Listbox
      v-model="wikiDataEntity"
      :options="search_result"
    />
  </div>
  </div>



  <div class="table p-text-center" v-if="data.length > 0">
    <DataTable :value="data" responsiveLayout="scroll">
      <Column field="title" header="Title"></Column>
      <Column
        v-for="col of columns"
        :field="col.field"
        :header="col.header"
        :key="col.field"
      >
        <template #body="slotProps">
          <i
            v-if="slotProps.data[slotProps.column.props.field]"
            class="pi pi-check"
          ></i>
          <i v-else class="pi pi-times"></i>
        </template>
      </Column>
    </DataTable>
  </div>

  <ProgressSpinner
    style="width: 50px; height: 50px; position: absolute; top: 10px; left: 10px"
    strokeWidth="8"
    v-if="isLoading"
  />
</template>

<script >
import { defineComponent, ref, watch } from "vue";

export default defineComponent({
  name: "Messenger",
  components: {},
  setup() {
    let isLoading = ref(false);
    let name = ref("");
    let search_result = ref([]);
    let wikiDataEntity = ref("");

    class SPARQLQueryDispatcher {
      constructor(endpoint) {
        this.endpoint = endpoint;
      }

      query(sparqlQuery) {
        const fullUrl =
          this.endpoint + "?query=" + encodeURIComponent(sparqlQuery);
        const headers = { Accept: "application/sparql-results+json" };

        return fetch(fullUrl, { headers }).then((body) => body.json());
      }
    }

    const data = ref([
      // {
      //   no: 0,
      //   title: "Testing Title",
      //   wd: 0,
      //   gs: 1,
      //   semscholar: 1,
      //   orcid: 1,
      //   viaf: 0,
      //   dblp: 0,
      //   dim: 1,
      //   msacademic: 0,
      //   dnbgnb: 1,
      //   acm: 0,
      //   crossref: 1,
      // },
    ]);

    const query = async () => {
      data.value = []
      data.value = await fetch(
        "http://127.0.0.1:5000/" + wikiDataEntity.value
      ).then((response) => {
        if (response.ok == false) {
          throw new Error(
            `Action on \`${response.url}\` failed: \`${response.status}\` \`${response.statusText}\`.`
          );
        }
        return response
          .json()
          .then((data) => Object.values(Object.values(data)[0]));
      });
    };

    const search = async () => {
      isLoading.value = true;
      search_result.value = []
      data.value = []
      const endpointUrl = "https://query.wikidata.org/sparql";
      const sparqlQuery =
        `SELECT ?entity WHERE {?entity rdfs:label "` + name.value + `"@en.}`;
      const queryDispatcher = new SPARQLQueryDispatcher(endpointUrl);
      queryDispatcher.query(sparqlQuery).then((data) => {
        let search_result_array = data.results.bindings.map((entry) => {
          const split = entry.entity.value.split("/");
          return split[split.length - 1];
        });
        if (search_result_array.length == 1) {
          wikiDataEntity.value = search_result_array[0]
        } else {
          search_result.value = search_result_array
          isLoading.value = false;
        }
      });
    };

    watch(
      () => wikiDataEntity.value,
      (newVal, oldVal) => {
        isLoading.value = true;
        query().then(() => {
          isLoading.value = false;
          // data.value = d;
        });
      }
    );

    const columns = ref([
      // { field: "no", header: "#" },
      // { field: "title", header: "Title" }, // do as hover tooltip
      { field: "Wikidata", header: "Wikidata" },
      { field: "Google Scholar", header: "Google Scholar" },
      { field: "Semantic Scholar", header: "Semantic Scholar" },
      { field: "ORCID", header: "ORCID" },
      { field: "VIAF", header: "VIAF" },
      { field: "DBLP", header: "DBLP" },
      { field: "Dimensions", header: "Dimensions" },
      { field: "Microsoft Academic", header: "MS Academic " },
      { field: "DNB/GNB", header: "DNB/GNB" },
      { field: "ACM Digital Library", header: "ACM DL" },
      // { field: "crossref", header: "CrossRef" },
    ]);
    return {
      name,
      search_result,
      wikiDataEntity,
      query,
      search,
      data,
      columns,
      isLoading,
    };
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.p-grid {
  margin: 5px;
}
.p-inputgroup {
  width: 50%;
}
.table {
  padding: 25px;
}
.size {
  width: 50px;
}
</style>
