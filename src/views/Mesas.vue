<template>
  <div class="view">
    <div class="head">
      <div class="title">Consulta de mesas</div>
      <div class="search">
        <SearchBox placeholder="Ingrese un numero de mesa a buscar" v-model="searchText"/>
      </div>
      <div class="buttons">
        <LargeButton img="table.svg" color="#54A7E3" text="Crear mesa" @click="goCrear"/>
        <LargeButton img="group-table.svg" color="#ED994C" text="Mesa conjunta"/>
      </div>
    </div>
    <div v-if="mesas" class="table">
      <GenericTable :headers="headers" :items="mesas"/>
    </div>
  </div>
</template>

<script>
import LargeButton from "../components/LargeButton";
import SearchBox from "../components/SearchBox";
import GenericTable from "../components/GenericTable";
import { api } from "../api";
import { storage } from "../localStorage";

export default {
  name: "mesas",

  data: () => ({
    searchText: "",
    mesas: undefined,
    headers: [
      {
        title: "Numero",
        prop: "id_Mesa"
      },
      {
        title: "Descripción",
        prop: "Desc_Mesa"
      },

      {
        title: "Estado",
        prop: "active"
      },
      {
        title: "Acción",
        prop: "actions"
      }
    ]
  }),
  methods: {
    goCrear() {
      this.$router.push({ path: "nueva-mesa" });
    },
    async load() {
      try {
        const { data } = await api.get("mesa");
        this.mesas = data.map(mesa => {
          const id = mesa.id_Mesa;
          const mesaConfig = storage.get("mesa-" + id) || {
            active: "Desocupada"
          };
          console.log(mesaConfig);
          if (mesaConfig.active == "Desocupada") {
            mesaConfig.actions = [
              {
                color: "red",
                onClick: async () => {
                  this.$swal({
                    title: "¿Estas seguro?",
                    text: "Esto no se podra revertir",
                    type: "warning",
                    showCancelButton: true,
                    confirmButtonColor: "#3085d6",
                    cancelButtonColor: "#d33",
                    confirmButtonText: "Eliminar"
                  }).then(async result => {
                    if (result.value) {
                      await api.delete(`mesa/${mesa.id_Mesa}`);
                      this.load();
                      this.$swal(
                        "Eliminado",
                        "La mesa ha sido eliminada",
                        "success"
                      );
                    }
                  });
                },
                tooltip: "Eliminar",
                img: "rubbish-bin.png"
              },
              {
                color: "#ED994C",
                onClick: () => {
                  this.$router.push(`mesas/${mesa.id_Mesa}`);
                },
                tooltip: "Actualizar",
                img: "pencil-edit-button.png"
              },
              {
                color: "#54A7E3",
                onClick: () => {
                  storage.set("mesa-" + id, {
                    active: "Ocupada"
                  });
                  console.log(
                    storage.get("mesa-" + id, {
                      active: "Ocupada"
                    })
                  );
                  this.$router.push({ path: `/editar-orden/${mesa.id_Mesa}` });
                },
                tooltip: "Nueva orden",
                img: "edit.png"
              }
            ];
          } else {
            mesaConfig.actions = [
              {
                color: "#54A7E3",
                onClick: () => {
                  storage.set("mesa-" + id, {
                    active: "Ocupada"
                  });
                  console.log(
                    storage.get("mesa-" + id, {
                      active: "Ocupada"
                    })
                  );
                  this.$router.push({ path: `/editar-orden/${mesa.id_Mesa}` });
                },
                tooltip: "Nueva orden",
                img: "edit.png"
              },
              {
                color: "#12CD25",
                onClick: () => {
                  storage.set("mesa-" + id, {
                    active: "Desocupada"
                  });
                  this.$router.push({ path: `/cobro-cuenta` });
                },
                tooltip: "Cobrar",
                img: "point-of-sale-terminal-pos.png"
              }
            ];
          }

          return { ...mesaConfig, ...mesa };
        });
      } catch (e) {
        console.log(e);
      }
    }
  },

  async mounted() {
    this.load();
  },
  components: {
    LargeButton,
    SearchBox,
    GenericTable
  }
};
</script>

<style scoped>
.view {
  padding: 30px 0px;
}
.head {
  display: flex;
  flex-direction: row;
}
.title {
  flex: 0.7;
  font-family: Roboto;
  font-style: normal;
  font-weight: normal;
  font-size: 25px;
  line-height: 29px;
  text-align: left;
  color: #000000;
  padding-left: 70px;
  display: flex;
  align-items: center;
}
.search {
  flex: 1.5;
}
.search > * {
  width: 100%;
}
.buttons {
  display: flex;
  flex-direction: row;
  flex: 1 0;
  justify-content: center;
  box-sizing: border-box;
}
.buttons > * {
  height: 40px;
  margin-left: 20px;
}

.table {
  margin-top: 40px;
}
.table > * {
  min-width: 1200px;
  margin: auto;
  text-align: center;
}
</style>