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
    <div
      v-if="search_result.length !== 0"
      class="p-text-center p-col-6 p-row p-offset-3"
    >
      <Listbox
        v-model="wikiDataEntity"
        optionLabel="bio"
        :options="search_result"
        :disabled="isLoading"
        :listStyle=" data.length > 0 ? `max-height: 10vh` : undefined"
      />
      <!-- <Card v-for="entry in search_result" :key="entry.entity">
        <template #content>
          {{ entry.bio }}
        </template>
      </Card> -->
    </div>
  </div>

  <div class="table p-text-center" v-if="data.length > 0">
    <DataTable :value="data" :scrollable="true" scrollHeight="65vh">
      <Column field="title" header="Title" style="min-width: 20vw"></Column>
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
    style="
      width: 50px;
      height: 50px;
      position: absolute;
      top: 20px;
      right: 20px;
    "
    strokeWidth="8"
    v-if="isLoading"
  />
   <Toast position="top-center" />
</template>

<script >
import { defineComponent, ref, watch } from "vue";
import { useToast } from "primevue/usetoast";

export default defineComponent({
  name: "Messenger",
  components: {},
  setup() {
    const toast = useToast();
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
      const queryEndpoint = "http://km.aifb.kit.edu/services/refbeebackend/";
      data.value = [];
      data.value = await fetch(
        queryEndpoint + wikiDataEntity.value.entity
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
      if (data.value.length ==0) {
        toast.add({
              severity: "error",
              summary: "No records anywhere!",
              detail: `Did not find any records anywhere of ${name.value}, the ${wikiDataEntity.value.bio}.`,
              life: 5000,
            });
      }
    };

    const search = async () => {
      isLoading.value = true;
      search_result.value = [];
      data.value = [];

      const endpointUrl = "https://query.wikidata.org/sparql";
      const sparqlQuery = `SELECT ?entity ?bio ?img 
          WHERE {
            ?entity rdfs:label "${name.value}"@en;
                    schema:description ?bio .
            OPTIONAL { ?entity wdt:P18 ?img }
            FILTER(lang(?bio)='en')
          }`;

      const queryDispatcher = new SPARQLQueryDispatcher(endpointUrl);
      queryDispatcher.query(sparqlQuery).then((data) => {
        const bindings = data.results.bindings.map((entry) => {
          const split = entry.entity.value.split("/");
          const entity = split[split.length - 1];
          const bio = entry.bio.value;
          const img = entry.img ? entry.img.value : undefined;
          return { entity, bio, img };
        });
        search_result.value = bindings;
        if (bindings.length == 1) {
          wikiDataEntity.value = bindings[0];
          toast.add({
              severity: "info",
              summary: "Loading...",
              detail: `Loading records of ${name.value} (WikiData ${wikiDataEntity.value.entity})`,
              life: 5000,
            });
        } else {
          isLoading.value = false;
        }
        if (bindings.length == 0) {
           toast.add({
              severity: "error",
              summary: "Who is this?",
              detail: `Did not find any ${name.value} at WikiData...`,
              life: 5000,
            });
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

.pi-check {
  color: var(--primary-color);
}
.pi-times {
  color: crimson;
}
</style>
