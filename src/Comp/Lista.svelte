<script>
  import { each } from "svelte/internal";

  let Fotografos = [];
  let Contador = "";
  let Dataset = [];

  /* Actualización automática de la lista */
  dbUsers.onSnapshot((docs) => {
    Fotografos = [];
    docs.forEach((doc) => {
      if (doc.data().Ciudad) Dataset.push(doc.data().Ciudad.capitalize());
      if (doc.data().Provincia) Dataset.push(doc.data().Provincia.capitalize());
      Fotografos = [...Fotografos, doc.data()];
    });
    Contador = Fotografos.length;
    Fotografos = [...Fotografos].sort(() => 0.5 - Math.random());
    Dataset = [...new Set(Dataset)];
  });

  // Filtrar la tabla
  function filtrarTabla() {
    const tableBody = document.querySelector("#tbody");
    if (!tableBody) return;

    const input = document.getElementById("buscar");
    const filas = document.getElementById("tbody").childNodes;

    function filtrar() {
      const query = new RegExp(input.value, "ig");

      Contador = 0;
      [...filas].map((fila) => {
        // texto a buscar
        const str = `${fila.dataset.nombre} ${fila.dataset.ciudad} ${fila.dataset.provincia} ${fila.dataset.pais} ${fila.dataset.email} ${fila.dataset.tel}`;
        Contador++;
        if (query.test(str)) {
          fila.style.display = "table-row";
        } else {
          fila.style.display = "none";
          Contador--;
        }
      });
    }

    if (input) {
      input.addEventListener("input", filtrar);
      input.focus();
    }
  }

  function mostrarInfo(fila) {
    const el = fila.target.parentElement;
    const data = el.dataset;
    const modal = document.querySelector(".modal");

    modal.style.display = "block";

    // Nombre
    modal.childNodes[0].textContent = data.nombre || "";

    // Website
    modal.childNodes[2].textContent = data.website;
    modal.childNodes[2].href = `//${data.website}`;

    // Email
    modal.childNodes[4].textContent = data.email;
    modal.childNodes[4].href = `mailto:${data.email}`;

    // Teléfono
    modal.childNodes[6].textContent = data.tel ? `Tel: ${data.tel}` : "";
    modal.childNodes[6].href = `tel:${data.tel}`;

    // Ciudad y Provincia
    modal.childNodes[8].textContent = `${data.ciudad}, ${data.provincia}`;
  }
  function cerrarModal() {
    const modal = document.querySelector(".modal");
    modal.style.display = "none";
  }

  // Capitalize the first letter of each word of a given string
  // Usage: 'capItalezE aLl fIrSt leTTers'.capitalize()
  String.prototype.capitalize = function () {
    if (!this || !typeof this === "string") return this;
    return this.trim()
      .split(" ")
      .reduce(
        (text, word) => text + " " + word[0].toUpperCase() + word.substr(1),
        ""
      );
  };

  /* Actualización por única vez 
  dbUsers.get().then((docs) => {
    docs.forEach((doc) => {
      Fotografos = [
        ...Fotografos,
        {
          Nombre:
            doc.data()["Nombre Comercial"] ||
            `${doc.data().Nombre} ${doc.data().Apellido}`,
          Email: doc.id,
          Tel: doc.data()["Teléfono"] || "",
          Ciudad: doc.data().Ciudad || doc.data().Provincia || "",
          Provincia: doc.data().Provincia || doc.data().Provincia || "",
          País: doc.data().País,
        },
      ];
    });
  });
*/
</script>

<style>
  input {
    font-size: 14px;
    width: 100%;
  }

  label {
    margin: 0;
    display: block;
    text-align: center;
  }

  table th {
    font-size: 14px;
    font-weight: 400;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    text-align: left;
    padding: 0.5em 0;
  }

  table th:nth-child(1) {
    width: 60%;
  }

  table th:nth-child(2) {
    width: 40%;
  }

  table tbody {
    cursor: pointer;
  }

  .modal {
    display: none;
    text-align: center;
    position: fixed;
    min-width: 300px;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    padding: 1rem;
    border-radius: 5px;
    box-shadow: 0 0 3px 0 grey;
  }
  .modal > * {
    display: block;
  }
  .modal > a:hover {
    color: var(--text);
  }
  .modal h1 {
    font-size: 1.2rem;
  }
  .modal button {
    margin: auto;
    line-height: inherit;
    border-color: white;
  }
</style>

<section>
  {#if Fotografos.length}
    <label for="buscar">{`${Contador} fotógrafos`}</label>
    <input
      on:input={filtrarTabla}
      id="buscar"
      type="search"
      list="datalist"
      class="flat"
      placeholder="Buscá fotógrafos de tu ciudad" />

    <datalist id="datalist">
      {#each Dataset as item}
        <option value={item} />
      {/each}
    </datalist>

    <table id="fotografos">
      <thead>
        <tr>
          <th>Nombre</th>
          <th>Ciudad</th>
        </tr>
      </thead>

      <tbody id="tbody">
        {#each Fotografos as item}
          <tr
            data-nombre={item['Nombre Comercial'] || `${item.Nombre} ${item.Apellido}`}
            data-email={item.Email}
            data-website={item.Website}
            data-tel={item.Teléfono || ''}
            data-ciudad={item.Ciudad || ''}
            data-provincia={item.Provincia}
            on:click={mostrarInfo}>
            <th>
              {item['Nombre Comercial'] || `${item.Nombre} ${item.Apellido}`}
            </th>
            <th>{item.Ciudad || item.Provincia}</th>

          </tr>
        {/each}
      </tbody>
    </table>
  {/if}
  <div class="modal Inv ">
    <h1>Nombre</h1>
    <a href target="_blank">Website</a>
    <a href>Email</a>
    <a href>Tel</a>
    <span>Ciudad y Provincia</span>
    <hr class="Sep" />
    <button class="Sol Inv" on:click={cerrarModal}>CERRAR</button>
  </div>
</section>
