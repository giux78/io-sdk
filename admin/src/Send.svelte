<script>
  export let key = "";

  import { Link } from "svelte-routing";
  import { formData } from "./store";
  import { onMount } from "svelte";

  let base = "http://localhost:3280/api/v1/web/guest/";

  let state = {};
  let action = "util/send";
  let data = {
    dest: "",
    subject: "",
    markdown: ""
  };

  async function start() {
    if (key == "") return;
    let id = window.atob(key);
    console.log("start");
    fetch(base + "util/cache", {
      method: "POST",
      body: JSON.stringify({ get: id }),
      headers: { "Content-Type": "application/json" }
    })
      .then(async res => {
        if (res.ok) {
          let message = await res.json();
          //console.log(data[id])
          data.dest = message[id].fiscal_code;
          data.subject = message[id].subject;
          data.markdown = message[id].markdown;
        }
      })
      .catch(err => {
        state = { error: err };
      });
  }
  onMount(start);

  function submitForm() {
    let url = base + action;
    console.log(url);
    fetch(url, {
      method: "POST",
      body: JSON.stringify(data),
      headers: {
        "Content-Type": "application/json"
      }
    })
      .then(async function(res) {
        state.result = await res.text();
      })
      .catch(function(err) {
        state.error = err.message;
      });
  }

  function resetForm() {
    result = "";
    error = "";
    for (key in data) {
      data[key] = "";
    }
  }
</script>

<h2>Send a Single Message</h2>
<br />
{#if state.result}
  <div class="alert alert-success" role="alert">
    <pre>{state.result}</pre>
  </div>
  <div>
    <button type="button" class="btn btn-primary" on:click={resetForm}>
      Nuovo Messaggio
    </button>
  </div>
{:else if state.error}
  <div class="alert alert-danger" role="alert">{state.error}</div>
  <div>
    <button type="button" class="btn btn-primary" on:click={resetForm}>
      Nuovo Messaggio
    </button>
  </div>
{:else}
  <div>
    <div class="form-group">
      <label class="active" for="codFiscDest">
        Codice Fiscale Destinatario
      </label>
      <input
        type="text"
        class="form-control"
        id="codFiscDest"
        bind:value={data.dest} />
    </div>
    <div class="form-group">
      <label class="active" for="messageSubject">Soggetto del messaggio</label>
      <input
        type="text"
        class="form-control"
        id="messageSubject"
        bind:value={data.subject} />
    </div>
    <div class="form-group">
      <textarea id="message" rows="3" bind:value={data.markdown} />
      <label class="active" for="message">Markdown del messaggio</label>
    </div>
    <div class="form-group">
      <div class="bootstrap-select-wrapper">
        <label>Endpoint</label>
        <select bind:value={action} title="Scegli una opzione">
          <option value="util/send">Development (Local)</option>
          <option value="iosdk/send">Production</option>
        </select>
      </div>
    </div>
    <div class="form-group">
      <button type="button" class="btn btn-primary" on:click={submitForm}>
        Send
      </button>
      <button type="button" class="btn btn-primary" on:click={resetForm}>
        Reset
      </button>
    </div>
  </div>
{/if}
