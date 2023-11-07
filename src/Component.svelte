<script>
  import { getContext, onDestroy } from "svelte"

  export let delta
  export let html
  export let label
  export let url

  export let deltaFieldApi;
  export let deltaFieldState;
  export let deltaFieldSchema;

  export let htmlFieldApi;
  export let htmlFieldState;
  export let htmlFieldSchema;

  let iframe

  const { styleable } = getContext("sdk")
  const component = getContext("component")
  const formContext = getContext('form')
  const formStepContext = getContext('form-step')

  const formApi = formContext?.formApi
  $: formStep = formStepContext ? $formStepContext || 1 : 1;
  $: deltaField = formApi?.registerField(delta, 'string', '', false, null, formStep)
  $: htmlField = formApi?.registerField(html, 'string', '', false, null, formStep)
  
  $: deltaFieldUnsubscribe = deltaField?.subscribe((value) => {
    deltaFieldState = value?.fieldState;
    deltaFieldApi = value?.fieldApi;
    deltaFieldSchema = value?.fieldSchema
  })

  $: htmlFieldUnsubscribe = htmlField?.subscribe((value) => {
    htmlFieldState = value?.fieldState;
    htmlFieldApi = value?.fieldApi;
    htmlFieldSchema = value?.fieldSchema
  })

  const handleMessage = e => {
    try {
      const { type, value } = e.data
      switch (type) {
        case 'state': {
          const { delta, html } = value
          if (delta && html) {
            deltaFieldApi.setValue(JSON.stringify(delta))
            htmlFieldApi.setValue(html)
          }
        }
        break;
        case 'ready': {
          if (deltaFieldState.value) {
            iframe.contentWindow.postMessage({ type: 'state', value: { delta: JSON.parse(deltaFieldState.value) } }, '*')
          }
        }
        break;
      }
    } catch(e) {
      console.warn(e)
    }
  }
  
  onDestroy(() => {
    deltaFieldApi?.deregister();
    htmlFieldApi?.deregister();
    deltaFieldUnsubscribe?.()
    htmlFieldUnsubscribe?.()
  })
</script>

<svelte:window on:message={handleMessage} />

<div use:styleable={$component.styles} class="spectrum-Form-Item">
  <label class="spectrum-FieldLabel spectrum-Form-itemLabel spectrum-FieldLabel--left spectrum-FieldLabel--sizeM">{label}</label>
  <div class="spectrum-Form-itemField">
    <iframe bind:this={iframe} src={url} height="600" style="width: 100%; height: 400px;" title="quill" frameborder="0" />
  </div>
</div>
