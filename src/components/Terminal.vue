<script setup lang="ts">
import { ref, onMounted, nextTick, reactive } from 'vue';

interface Project {
  name: string;
  description: string;
}

interface FetchInfo {
  user: string;
  os: string;
  host: string;
  kernel: string;
  shell: string;
  faith: string;
  languages: string;
}

interface InitialContent {
  fetch: FetchInfo;
  about: string;
  faith: string;
  projects: Project[];
  logo: string;
}

const props = defineProps<{
  initialContent: InitialContent;
}>();

const input = ref('');
const history = reactive<string[]>([]);
const output = reactive<{ type: 'command' | 'result', content: string | any, isRaw?: boolean }[]>([]);
const inputRef = ref<HTMLInputElement | null>(null);
const terminalRef = ref<HTMLElement | null>(null);
const historyIndex = ref(-1);

const commands = ['help', 'about', 'faith', 'projects', 'fetch', 'clear', 'socials'];

const executeCommand = (cmd: string) => {
  const trimmedCmd = cmd.trim().toLowerCase();
  output.push({ type: 'command', content: `guest@gigio.dev:~$ ${cmd}` });
  
  if (trimmedCmd === '') return;
  
  history.unshift(cmd);
  historyIndex.value = -1;

  switch (trimmedCmd) {
    case 'help':
      output.push({ type: 'result', content: `Available commands: ${commands.join(', ')}` });
      break;
    case 'about':
      output.push({ type: 'result', content: props.initialContent.about });
      break;
    case 'faith':
      output.push({ type: 'result', content: props.initialContent.faith });
      break;
    case 'projects':
      let projList = props.initialContent.projects.map(p => `- ${p.name}: ${p.description}`).join('\n');
      output.push({ type: 'result', content: projList });
      break;
    case 'fetch':
      output.push({ type: 'result', content: props.initialContent, isRaw: false });
      break;
    case 'clear':
      output.splice(0, output.length);
      break;
    case 'socials':
      output.push({ type: 'result', content: 'GitHub: https://github.com/gigiodelchiaro\nLinkedIn: https://linkedin.com/in/gigiodelchiaro' });
      break;
    default:
      output.push({ type: 'result', content: `Command not found: ${trimmedCmd}. Type 'help' for available commands.` });
  }

  nextTick(() => {
    scrollToBottom();
  });
};

const handleEnter = () => {
  executeCommand(input.value);
  input.value = '';
};

const handleKeyDown = (e: KeyboardEvent) => {
  if (e.key === 'ArrowUp') {
    e.preventDefault();
    if (historyIndex.value < history.length - 1) {
      historyIndex.value++;
      input.value = history[historyIndex.value];
    }
  } else if (e.key === 'ArrowDown') {
    e.preventDefault();
    if (historyIndex.value > 0) {
      historyIndex.value--;
      input.value = history[historyIndex.value];
    } else {
      historyIndex.value = -1;
      input.value = '';
    }
  } else if (e.key === 'Tab') {
    e.preventDefault();
    const matches = commands.filter(c => c.startsWith(input.value.toLowerCase()));
    if (matches.length === 1) {
      input.value = matches[0];
    }
  }
};

const scrollToBottom = () => {
  if (terminalRef.value) {
    terminalRef.value.scrollTop = terminalRef.value.scrollHeight;
  }
};

onMounted(() => {
  document.documentElement.classList.add('terminal-active');
  inputRef.value?.focus();
  // Auto-run fetch on load
  executeCommand('fetch');
});

const focusInput = () => {
  inputRef.value?.focus();
};
</script>

<template>
  <div class="terminal-container font-mono text-sm md:text-base bg-transparent min-h-[400px] flex flex-col" @click="focusInput" ref="terminalRef">
    <div class="output-area flex-grow">
      <div v-for="(line, index) in output" :key="index" class="mb-2 whitespace-pre-wrap">
        <template v-if="line.type === 'command'">
          <span class="text-[var(--terminal-green)] font-bold">{{ line.content }}</span>
        </template>
        <template v-else-if="line.isRaw === false && typeof line.content === 'object'">
          <div class="flex flex-col md:flex-row gap-4 my-4">
            <pre class="text-[var(--fedora-blue)] font-bold leading-tight">{{ props.initialContent.logo }}</pre>
            <div class="flex flex-col justify-center">
              <h1 class="text-xl font-bold text-[var(--fedora-blue)] mb-1">{{ line.content.user }}</h1>
              <hr class="border-[var(--text)] mb-1" />
              <p><span class="text-[var(--fedora-blue)] font-bold">OS:</span> {{ line.content.os }}</p>
              <p><span class="text-[var(--fedora-blue)] font-bold">Host:</span> {{ line.content.host }}</p>
              <p><span class="text-[var(--fedora-blue)] font-bold">Kernel:</span> {{ line.content.kernel }}</p>
              <p><span class="text-[var(--fedora-blue)] font-bold">Shell:</span> {{ line.content.shell }}</p>
              <p><span class="text-[var(--catholic-gold)] font-bold">Faith:</span> {{ line.content.faith }}</p>
              <p><span class="text-[var(--fedora-blue)] font-bold">Languages:</span> {{ line.content.languages }}</p>
            </div>
          </div>
        </template>
        <template v-else>
          <div class="text-[var(--text)]">{{ line.content }}</div>
        </template>
      </div>
    </div>
    
    <div class="input-line flex items-center">
      <span class="text-[var(--terminal-green)] font-bold mr-2">guest@gigio.dev:~$</span>
      <input 
        ref="inputRef"
        v-model="input"
        class="bg-transparent border-none outline-none flex-grow text-[var(--text)] caret-[var(--terminal-green)]"
        @keydown.enter="handleEnter"
        @keydown="handleKeyDown"
        spellcheck="false"
        autocomplete="off"
      />
    </div>
  </div>
</template>

<style scoped>
.terminal-container {
  overflow-y: auto;
  max-height: 80vh;
}

input {
  font-family: inherit;
  font-size: inherit;
}
</style>
