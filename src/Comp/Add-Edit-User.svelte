<script>
  import { loginUser } from "../store";

  let User = {};
  let Municipios;

  if (localStorage && localStorage.Email) User.Email = localStorage.Email;
  if ($loginUser) User.Email = $loginUser.email;

  function actualizaDatos(id) {
    dbUsers
      .doc(id)
      .get()
      .then((doc) => {
        if (doc.exists) {
          User = doc.data();
          User.Actualización = new Date();
        }
      });
  }

  if (User.Email) {
    // const email = auth.currentUser.email;
    const email = User.Email;
    dbUsers
      .doc(email)
      .get()
      .then((doc) => {
        if (doc.exists) {
          User = doc.data();
          User.Actualización = new Date();
        } else {
          console.log(`El ususario no existe en la base de datos y se creará`);
          const newUser = {
            id: LoginUser.uid,
            Email: LoginUser.email,
            Actualización: new Date(),
            Nombre: LoginUser.displayName.split(" ").slice(0, -1).join(" "),
            Apellido: LoginUser.displayName.split(" ").slice(-1).join(" "),
            Avatar: LoginUser.providerData[0].photoURL,
            "Nombre Completo": LoginUser.displayName,
          };
          if (LoginUser.phoneNumber) newUser.Teléfono = LoginUser.phoneNumber;
          dbUsers.doc(email).set(newUser);
          User = newUser;
        }
        // document.forms[0]["Email"].disabled = true;
      });
  }

  /* Auto Guardado de Datos en Firestore */
  function guardarDatos() {
    const form = document.forms[0];
    if (!form.checkValidity()) return false;
    const email = form["Email"].value;
    console.info("Guadando datos...");
    User["Actualización"] = new Date();
    dbUsers.doc(email).set(User);
  }

  async function chequearEmailValido() {
    const input = document.forms[0]["Email"];
    input.setCustomValidity("");
    const valido = input.validity.valid;

    if (!valido) return false;
    const email = input.value;

    // Guardo en Local
    if (localStorage) localStorage.Email = email;

    /* Chequea si existe el email */
    const esValido = await dbUsers
      .where("Email", "==", email)
      .get()
      .then((doc) => {
        if (!doc.empty) {
          // el email ya existe
          actualizaDatos(email);
          input.disabled = true;

          // const error = `${email} ya se existe !`;
          // console.error(error);
          // input.setCustomValidity(error);
          // input.placeholder = error;
          // input.value = "";
          // input.focus();
        }
      });
    return esValido;
  }

  async function cargaMunicipios(provincia) {
    Municipios = await obtenerMunicipios(provincia);
  }

  async function obtenerMunicipios(provincia = "") {
    if (provincia) provincia = `&provincia=${provincia}`;
    const url = `https://apis.datos.gob.ar/georef/api/municipios?max=5000${provincia}`;
    const fetchData = await fetch(url);
    const data = await fetchData.json();
    const reduce = data.municipios.reduce((accu, item) => {
      const municipio = item.nombre;
      const provincia = item.provincia.nombre;
      const geoloc = item.centroide;
      return [...accu, municipio];
    }, "");
    const results = [...new Set(reduce)];
    return results;
  }

  /*
  function uploadImagen() {
    const file = document.getElementById("imageFile").files[0];

    if (file) {
      const imagen = document.getElementById("avatar");
      imagen.src = URL.createObjectURL(file);
    }
  }
  */
</script>

<style>
  button {
    text-align: center;
  }
</style>

<section>
  <h1 class="title">Mis Datos</h1>

  <form on:change={guardarDatos} action="javascript:void(0);">
    <fieldset>
      <caption>Datos Personales</caption>

      <label for="Email">Email</label>
      <input
        on:change={chequearEmailValido}
        bind:value={User.Email}
        name="Email"
        type="email"
        required />

      <label for="Nombre">Nombre</label>
      <input bind:value={User.Nombre} name="Nombre" type="text" required />

      <label for="Apellido">Apeliido</label>
      <input bind:value={User.Apellido} name="Apellido" type="text" required />

      <label for="Teléfono">Teléfono</label>
      <input bind:value={User.Teléfono} name="Teléfono" type="tel" />

      <label for="Comercial Comercial">Nombre comercial</label>
      <input
        bind:value={User['Nombre Comercial']}
        name="Comercial Comercial"
        type="text"
        placeholder="Opcional" />

      <label for="Estudio">Nombre de su estudio:</label>
      <input
        bind:value={User.Estudio}
        name="Estudio"
        type="text"
        placeholder="Opcional" />

      <label for="País">País</label>
      <select bind:value={User['País']} name="País" required>
        <option value="Argentina">Argentina</option>
      </select>
      <label for="Provincia">Provincia</label>
      <select
        on:focus={(e) => cargaMunicipios(e.target.value)}
        on:input={(e) => cargaMunicipios(e.target.value)}
        bind:value={User.Provincia}
        name="Provincia"
        required>
        <option value="Buenos Aires">Bs. As.</option>
        <option value="Catamarca">Catamarca</option>
        <option value="Chaco">Chaco</option>
        <option value="Chubut">Chubut</option>
        <option value="Córdoba">Córdoba</option>
        <option value="Corrientes">Corrientes</option>
        <option value="Entre Ríos">Entre Ríos</option>
        <option value="Formosa">Formosa</option>
        <option value="Jujuy">Jujuy</option>
        <option value="La Pampa">La Pampa</option>
        <option value="La Rioja">La Rioja</option>
        <option value="Mendoza">Mendoza</option>
        <option value="Misiones">Misiones</option>
        <option value="Neuquen">Neuquen</option>
        <option value="Río Negro">Río Negro</option>
        <option value="Salta">Salta</option>
        <option value="San Juan">San Juan</option>
        <option value="San Luis">San Luis</option>
        <option value="Santa Cruz">Santa Cruz</option>
        <option value="Santa Fe">Santa Fe</option>
        <option value="Sgo. del Estero">Sgo. del Estero</option>
        <option value="Tierra del Fuego">Tierra del Fuego</option>
        <option value="Tucumán">Tucumán</option>
      </select>

      <label for="Ciudad">Ciudad</label>
      <input
        bind:value={User.Ciudad}
        name="Ciudad"
        type="text"
        list="Municipios" />
      {#if Municipios}
        <datalist id="Municipios" style="display: none">
          {#each Municipios as item}
            <option value={item} />
          {/each}
        </datalist>
      {/if}
    </fieldset>
    <hr class="Sep" />
    <hr class="Sep" />
    <fieldset>

      <caption>Redes sociales</caption>
      <label for="Website">Sitio Web (opcional)</label>
      <input
        bind:value={User.Website}
        name="Website"
        type="text"
        placeholder="Ej: misitio.com" />

      <label for="Facebook">Facebook (opcional)</label>
      <input
        bind:value={User.Facebook}
        name="Facebook"
        type="text"
        placeholder="Ej: fb.com/usario" />

      <label for="Instagram">Instagram (opcional)</label>
      <input
        bind:value={User.Instagram}
        name="Instagram"
        type="text"
        placeholder="Ej: instagram.com/usuario" />

      <label for="Twitter">Twitter (opcional)</label>
      <input
        bind:value={User.Twitter}
        name="Twitter"
        type="text"
        placeholder="Ej: twitter.com/galiprandi" />

      <label for="Información Adicional">Información Adicional</label>
      <textarea
        bind:value={User['Información Adicional']}
        name="Información Adicional"
        cols="30"
        rows="10"
        placeholder="Opcional" />
    </fieldset>
    <button on:click={guardarDatos} class="Inv" id="Guardar">
      Guardar Datos
    </button>
    <!-- svelte-ignore missing-declaration -->
    <button on:click={() => deleteUserById(User.Email)} class="Out">
      BORRAR MIS DATOS
    </button>

  </form>

</section>
