<template>
  <div class="p-grid">
    <div class="p-inputgroup p-col-6 p-row p-offset-3">
      <Button icon="pi pi-search" @click="query" />
      <InputText
        placeholder="WikiData Entity (e.g. Q92692)"
        v-model="wikiDataEntity"
        @keyup.enter="query"
      />
      <Button label="Search" @click="query" />
    </div>
  </div>

  <div class="table p-text-center">
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
</template>

<script >
import { defineComponent, ref } from "vue";

export default defineComponent({
  name: "Messenger",
  components: {},
  setup() {
    const wikiDataEntity = "Q92692";
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
      data.value = await fetch("http://127.0.0.1:5000/" + wikiDataEntity).then(
        (response) => {
          if (response.ok == false) {
            throw new Error(
              `Action on \`${response.url}\` failed: \`${response.status}\` \`${response.statusText}\`.`
            );
          }
          return response
            .json()
            .then((data) => Object.values(Object.values(data)[0]));
        }
      );
    };

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
      wikiDataEntity,
      query,
      data,
      columns,
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
