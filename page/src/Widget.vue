<template>
  <div>
    <div v-if="loading" class="d-flex justify-center pa-4">
      <v-progress-circular indeterminate size="24" width="2" />
    </div>
    <div v-else-if="!isConfigured" class="text-center pa-3">
      <v-icon size="32" color="grey" class="mb-2">mdi-cog-outline</v-icon>
      <div class="text-body-2 text-medium-emphasis mb-2">{{ $t('plugin_amdgpu_top.not_configured') }}</div>
      <v-btn size="small" variant="tonal" color="primary" :to="`/plugins/amdgpu-top`">
        <v-icon start size="16">mdi-open-in-app</v-icon>
        {{ $t('plugin_amdgpu_top.configure') }}
      </v-btn>
    </div>
    <div v-else>
      <div v-for="(gpu, index) in settings.gpus" :key="index" :class="{ 'mt-3': index > 0 }">
        <div class="d-flex align-center mb-2" style="min-width: 0">
          <span class="text-body-2 font-weight-bold" :title="getGpuName(gpu.pci)" style="white-space: nowrap; overflow: hidden; text-overflow: ellipsis; min-width: 0; flex: 1">{{ getGpuName(gpu.pci) }}</span>
          <v-chip size="x-small" variant="outlined" class="ml-2" style="flex-shrink: 0">{{ gpu.pci }}</v-chip>
        </div>
        <div v-if="gpuData[gpu.pci]">
          <div v-if="gpuData[gpu.pci].Sensors?.['Edge Temperature']?.value != null" class="d-flex align-center mb-1" style="gap: 6px">
            <span class="text-caption" style="width: 80px; flex-shrink: 0"><b>{{ $t('plugin_amdgpu_top.temperature') }}</b></span>
            <span class="text-caption" style="flex-shrink: 0">{{ gpuData[gpu.pci].Sensors['Edge Temperature'].value }} °C</span>
          </div>
          <div v-if="gpuData[gpu.pci].Sensors?.['Average Power']?.value != null" class="d-flex align-center mb-1" style="gap: 6px">
            <span class="text-caption" style="width: 80px; flex-shrink: 0"><b>{{ $t('plugin_amdgpu_top.power') }}</b></span>
            <span class="text-caption" style="flex-shrink: 0">{{ gpuData[gpu.pci].Sensors['Average Power'].value }} W</span>
          </div>
          <div v-if="gpuData[gpu.pci].Sensors?.['GFX_SCLK']?.value != null" class="d-flex align-center mb-1" style="gap: 6px">
            <span class="text-caption" style="width: 80px; flex-shrink: 0"><b>{{ $t('plugin_amdgpu_top.gfx_clock') }}</b></span>
            <span class="text-caption" style="flex-shrink: 0">{{ gpuData[gpu.pci].Sensors['GFX_SCLK'].value }} MHz</span>
          </div>
          <div v-if="gpuData[gpu.pci].Sensors?.['Fan']?.value != null" class="d-flex align-center mb-1" style="gap: 6px">
            <span class="text-caption" style="width: 80px; flex-shrink: 0"><b>{{ $t('plugin_amdgpu_top.fan') }}</b></span>
            <span class="text-caption" style="flex-shrink: 0">{{ gpuData[gpu.pci].Sensors['Fan'].value }} RPM</span>
          </div>
          <div v-if="gpuData[gpu.pci].VRAM" class="d-flex align-center mb-1" style="gap: 6px">
            <span class="text-caption" style="width: 80px; flex-shrink: 0"><b>VRAM</b></span>
            <v-progress-linear
              :model-value="getVramPercent(gpu.pci)"
              height="10"
              :color="getVramPercent(gpu.pci) >= 90 ? 'red' : getVramPercent(gpu.pci) >= 75 ? 'orange' : 'green'"
              style="border-radius: 5px; overflow: hidden; flex: 1"
            >
              <template #default>
                <span style="font-size: 9px">{{ getVramPercent(gpu.pci).toFixed(0) }}%</span>
              </template>
            </v-progress-linear>
          </div>
          <div v-if="gpuData[gpu.pci].VRAM?.['Total GTT']" class="d-flex align-center mb-1" style="gap: 6px">
            <span class="text-caption" style="width: 80px; flex-shrink: 0"><b>GTT</b></span>
            <v-progress-linear
              :model-value="getGttPercent(gpu.pci)"
              height="10"
              :color="getGttPercent(gpu.pci) >= 90 ? 'red' : getGttPercent(gpu.pci) >= 75 ? 'orange' : 'green'"
              style="border-radius: 5px; overflow: hidden; flex: 1"
            >
              <template #default>
                <span style="font-size: 9px">{{ getGttPercent(gpu.pci).toFixed(0) }}%</span>
              </template>
            </v-progress-linear>
          </div>
          <div v-if="gpuData[gpu.pci].gpu_activity?.GFX?.value != null" class="d-flex align-center mb-1" style="gap: 6px">
            <span class="text-caption" style="width: 80px; flex-shrink: 0"><small>GFX</small></span>
            <v-progress-linear
              :model-value="gpuData[gpu.pci].gpu_activity.GFX.value"
              height="10"
              :color="gpuData[gpu.pci].gpu_activity.GFX.value >= 90 ? 'red' : gpuData[gpu.pci].gpu_activity.GFX.value >= 75 ? 'orange' : 'green'"
              style="border-radius: 5px; overflow: hidden; flex: 1"
            >
              <template #default>
                <span style="font-size: 9px">{{ gpuData[gpu.pci].gpu_activity.GFX.value }}%</span>
              </template>
            </v-progress-linear>
          </div>
          <div v-if="gpuData[gpu.pci].gpu_activity?.MediaEngine?.value != null" class="d-flex align-center mb-1" style="gap: 6px">
            <span class="text-caption" style="width: 80px; flex-shrink: 0"><small>Media</small></span>
            <v-progress-linear
              :model-value="gpuData[gpu.pci].gpu_activity.MediaEngine.value"
              height="10"
              :color="gpuData[gpu.pci].gpu_activity.MediaEngine.value >= 90 ? 'red' : gpuData[gpu.pci].gpu_activity.MediaEngine.value >= 75 ? 'orange' : 'green'"
              style="border-radius: 5px; overflow: hidden; flex: 1"
            >
              <template #default>
                <span style="font-size: 9px">{{ gpuData[gpu.pci].gpu_activity.MediaEngine.value }}%</span>
              </template>
            </v-progress-linear>
          </div>
          <div class="mt-1">
            <div v-if="getProcessCount(gpu.pci) === 0" class="text-caption text-medium-emphasis">
              <v-icon size="12" class="mr-1">mdi-application-outline</v-icon>
              0 {{ $t('plugin_amdgpu_top.processes') }}
            </div>
            <details v-else>
              <summary style="cursor: pointer; color: var(--v-theme-primary); text-decoration: underline" class="text-body-2 mb-1">
                <v-icon size="12" class="mr-1">mdi-application-outline</v-icon>
                {{ getProcessCount(gpu.pci) }} {{ getProcessCount(gpu.pci) === 1 ? $t('plugin_amdgpu_top.process') : $t('plugin_amdgpu_top.processes') }}
              </summary>
              <div v-for="(proc, pid) in gpuData[gpu.pci].fdinfo" :key="pid" class="mb-2 ml-1">
                <div class="d-flex align-center mb-1">
                  <span class="text-caption"><b>{{ proc.name || 'Unknown' }}</b></span>
                  <span class="text-caption text-medium-emphasis ml-2">PID: {{ pid }}</span>
                </div>
                <div v-if="proc.usage?.usage">
                  <div v-if="proc.usage.usage.GFX" class="d-flex align-center mb-1" style="gap: 6px">
                    <span class="text-caption" style="width: 80px; flex-shrink: 0"><small>GFX</small></span>
                    <v-progress-linear
                      :model-value="proc.usage.usage.GFX.value || 0"
                      height="10"
                      :color="proc.usage.usage.GFX.value >= 90 ? 'red' : proc.usage.usage.GFX.value >= 75 ? 'orange' : 'green'"
                      style="border-radius: 5px; overflow: hidden; flex: 1"
                    >
                      <template #default>
                        <span style="font-size: 9px">{{ proc.usage.usage.GFX.value || 0 }}%</span>
                      </template>
                    </v-progress-linear>
                  </div>
                  <div v-if="proc.usage.usage.Compute" class="d-flex align-center mb-1" style="gap: 6px">
                    <span class="text-caption" style="width: 80px; flex-shrink: 0"><small>Compute</small></span>
                    <v-progress-linear
                      :model-value="proc.usage.usage.Compute.value || 0"
                      height="10"
                      :color="proc.usage.usage.Compute.value >= 90 ? 'red' : proc.usage.usage.Compute.value >= 75 ? 'orange' : 'green'"
                      style="border-radius: 5px; overflow: hidden; flex: 1"
                    >
                      <template #default>
                        <span style="font-size: 9px">{{ proc.usage.usage.Compute.value || 0 }}%</span>
                      </template>
                    </v-progress-linear>
                  </div>
                  <div v-if="proc.usage.usage.VRAM" class="d-flex align-center mb-1" style="gap: 6px">
                    <span class="text-caption" style="width: 80px; flex-shrink: 0"><small>VRAM</small></span>
                    <span class="text-caption">{{ proc.usage.usage.VRAM.value || 0 }} {{ proc.usage.usage.VRAM.unit }}</span>
                  </div>
                  <div v-if="proc.usage.usage.GTT" class="d-flex align-center mb-1" style="gap: 6px">
                    <span class="text-caption" style="width: 80px; flex-shrink: 0"><small>GTT</small></span>
                    <span class="text-caption">{{ proc.usage.usage.GTT.value || 0 }} {{ proc.usage.usage.GTT.unit }}</span>
                  </div>
                </div>
              </div>
            </details>
          </div>
        </div>
        <div v-else class="text-caption text-medium-emphasis text-center pa-2">
          {{ $t('plugin_amdgpu_top.loading') }}
        </div>
        <v-divider v-if="index < settings.gpus.length - 1" class="mt-2" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';

const loading = ref(true);
const settings = ref({ gpus: [], interval: 2 });
const gpuData = ref({});
const availableGpus = ref([]);
let pollInterval = null;

const isConfigured = computed(() => {
  return settings.value.gpus && settings.value.gpus.length > 0 && settings.value.gpus.some((g) => g.pci);
});

const getAuthHeaders = () => ({
  Authorization: 'Bearer ' + localStorage.getItem('authToken'),
});

const getGpuName = (pci) => {
  const gpu = availableGpus.value.find((g) => g.pci === pci);
  return gpu ? gpu.name : `GPU ${pci}`;
};

const getVramPercent = (pci) => {
  const data = gpuData.value[pci];
  if (!data?.VRAM) return 0;
  const total = data.VRAM['Total VRAM']?.value || 1;
  const used = data.VRAM['Total VRAM Usage']?.value || 0;
  return (used / total) * 100;
};

const getGttPercent = (pci) => {
  const data = gpuData.value[pci];
  if (!data?.VRAM) return 0;
  const total = data.VRAM['Total GTT']?.value || 1;
  const used = data.VRAM['Total GTT Usage']?.value || 0;
  return (used / total) * 100;
};

const getProcessCount = (pci) => {
  const fdinfo = gpuData.value[pci]?.fdinfo;
  if (!fdinfo) return 0;
  return Object.keys(fdinfo).length;
};

const fetchSettings = async () => {
  try {
    const res = await fetch('/api/v1/mos/plugins/settings/amdgpu-top', {
      headers: getAuthHeaders(),
    });
    if (res.ok) {
      const data = await res.json();
      settings.value = {
        gpus: Array.isArray(data.gpus) ? data.gpus : [],
        interval: data.interval || 2,
      };
    }
  } catch { }
};

const fetchAvailableGpus = async () => {
  try {
    const res = await fetch('/api/v1/system/gpus', {
      headers: getAuthHeaders(),
    });
    if (res.ok) {
      const data = await res.json();
      if (data.AMD && Array.isArray(data.AMD)) {
        availableGpus.value = data.AMD.map((gpu) => ({
          pci: gpu.pci,
          name: gpu.name,
        }));
      }
    }
  } catch { }
};

const fetchGpuData = async (gpu) => {
  try {
    const res = await fetch('/api/v1/mos/plugins/query', {
      method: 'POST',
      headers: {
        ...getAuthHeaders(),
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        command: 'amdgpu_top',
        args: ['-n', '1', '-J', '-i', gpu.index?.toString() || '0'],
        timeout: 5,
        parse_json: true,
      }),
    });

    if (res.ok) {
      const data = await res.json();
      const output = data.output;
      if (output?.devices && output.devices.length > 0) {
        gpuData.value[gpu.pci] = output.devices[0];
      }
    }
  } catch { }
};

const fetchAllGpuData = async () => {
  if (!isConfigured.value) return;
  for (const gpu of settings.value.gpus) {
    if (gpu.pci) {
      await fetchGpuData(gpu);
    }
  }
};

const startPolling = () => {
  stopPolling();
  if (!isConfigured.value) return;
  fetchAllGpuData();
  const interval = Math.max(2, settings.value.interval) * 1000;
  pollInterval = setInterval(fetchAllGpuData, interval);
};

const stopPolling = () => {
  if (pollInterval) {
    clearInterval(pollInterval);
    pollInterval = null;
  }
};

onMounted(async () => {
  try {
    await Promise.all([fetchSettings(), fetchAvailableGpus()]);
    if (isConfigured.value) {
      startPolling();
    }
  } catch { }
  finally {
    loading.value = false;
  }
});

onUnmounted(() => {
  stopPolling();
});
</script>
