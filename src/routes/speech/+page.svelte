

<script>
import { browser } from '$app/environment';
  let isSpeechSupported =browser && 'speechSynthesis' in window;
  let textToSpeak = "le";
  function speak() {
    if(browser){
        const utterance = new SpeechSynthesisUtterance(textToSpeak);
        speechSynthesis.speak(utterance);
    }
  }
</script>

{#if isSpeechSupported}
  <button on:click={speak}>Speak</button>
{:else}
  <p>Text-to-speech is not supported in this browser.</p>
{/if}

