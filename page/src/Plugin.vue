<template>
  <div>
    <h2 class="mb-4">amdgpu TOP Plugin</h2>
    <v-skeleton-loader v-if="loading" :loading="true" type="card" />
    <v-card v-else-if="!isConfigured" class="mb-4 pa-0">
      <v-card-text class="pa-4">
        Please configure the plugin first to display GPU data.
      </v-card-text>
    </v-card>
    <div v-else style="margin-bottom: 80px">
      <v-card v-for="(gpu, index) in settings.gpus" :key="index" class="mb-4 pa-0">
        <v-card-title class="d-flex align-center">
          <span>{{ getGpuName(gpu.pci) }}</span>
          <v-spacer />
          <v-chip size="small">{{ gpu.pci }}</v-chip>
        </v-card-title>
        <v-card-text v-if="gpuData[gpu.pci]" class="pa-4">
          <v-row dense>
            <v-col v-if="gpuData[gpu.pci].Info?.['Chip Class']" cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Chip Class</strong></div>
              <div class="text-body-2">{{ gpuData[gpu.pci].Info['Chip Class'] }}</div>
            </v-col>
            <v-col v-if="gpuData[gpu.pci].Info?.['GPU Type']" cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>GPU Type</strong></div>
              <div class="text-body-2">{{ gpuData[gpu.pci].Info['GPU Type'] }}</div>
            </v-col>
            <v-col v-if="gpuData[gpu.pci].Info?.['VRAM Type']" cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>VRAM Type</strong></div>
              <div class="text-body-2">{{ gpuData[gpu.pci].Info['VRAM Type'] }}</div>
            </v-col>
            <v-col v-if="gpuData[gpu.pci].Info?.['Total Compute Unit']" cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Compute Units</strong></div>
              <div class="text-body-2">{{ gpuData[gpu.pci].Info['Total Compute Unit'] }}</div>
            </v-col>
          </v-row>
          <v-divider class="mt-2 mb-2" />
          <v-row dense>
            <v-col v-if="gpuData[gpu.pci].Sensors?.['Edge Temperature']?.value != null" cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Temperature</strong></div>
              <div class="text-body-2">{{ gpuData[gpu.pci].Sensors['Edge Temperature'].value }} °C</div>
            </v-col>
            <v-col v-if="gpuData[gpu.pci].Sensors?.['Junction Temperature']?.value != null" cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Junction Temp</strong></div>
              <div class="text-body-2">{{ gpuData[gpu.pci].Sensors['Junction Temperature'].value }} °C</div>
            </v-col>
            <v-col v-if="gpuData[gpu.pci].Sensors?.['Average Power']?.value != null" cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Average Power</strong></div>
              <div class="text-body-2">{{ gpuData[gpu.pci].Sensors['Average Power'].value }} W</div>
            </v-col>
            <v-col v-if="gpuData[gpu.pci].Sensors?.['Input Power']?.value != null" cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Input Power</strong></div>
              <div class="text-body-2">{{ gpuData[gpu.pci].Sensors['Input Power'].value }} W</div>
            </v-col>
            <v-col v-if="gpuData[gpu.pci].Sensors?.['GFX_SCLK']?.value != null" cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>GFX Clock</strong></div>
              <div class="text-body-2">{{ gpuData[gpu.pci].Sensors['GFX_SCLK'].value }} MHz</div>
            </v-col>
            <v-col v-if="gpuData[gpu.pci].Sensors?.['GFX_MCLK']?.value != null" cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Memory Clock</strong></div>
              <div class="text-body-2">{{ gpuData[gpu.pci].Sensors['GFX_MCLK'].value }} MHz</div>
            </v-col>
            <v-col v-if="gpuData[gpu.pci].Sensors?.['Fan']?.value != null" cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Fan Speed</strong></div>
              <div class="text-body-2">{{ gpuData[gpu.pci].Sensors['Fan'].value }} RPM</div>
            </v-col>
            <v-col v-if="gpuData[gpu.pci].Sensors?.['Power Profile']" cols="6" md="3">
              <div class="text-caption text-medium-emphasis"><strong>Power Profile</strong></div>
              <div class="text-body-2">{{ gpuData[gpu.pci].Sensors['Power Profile'] }}</div>
            </v-col>
          </v-row>
          <v-divider class="mt-2 mb-2" />
          <v-row v-if="gpuData[gpu.pci].VRAM" dense>
            <v-col cols="12" md="6">
              <div class="d-flex align-center mb-1">
                <span class="text-caption" style="width: 80px"><strong>VRAM:</strong></span>
                <v-progress-linear
                  :model-value="getVramPercent(gpu.pci)"
                  height="16"
                  :color="getVramPercent(gpu.pci) >= 90 ? 'red' : getVramPercent(gpu.pci) >= 75 ? 'orange' : 'green'"
                  style="border-radius: 7px; overflow: hidden; flex: 1"
                >
                  <template #default>
                    <span><small>{{ gpuData[gpu.pci].VRAM['Total VRAM Usage']?.value || 0 }} / {{ gpuData[gpu.pci].VRAM['Total VRAM']?.value || 0 }} MiB</small></span>
                  </template>
                </v-progress-linear>
              </div>
            </v-col>
            <v-col cols="12" md="6">
              <div class="d-flex align-center mb-1">
                <span class="text-caption" style="width: 80px"><strong>GTT:</strong></span>
                <v-progress-linear
                  :model-value="getGttPercent(gpu.pci)"
                  height="16"
                  :color="getGttPercent(gpu.pci) >= 90 ? 'red' : getGttPercent(gpu.pci) >= 75 ? 'orange' : 'green'"
                  style="border-radius: 7px; overflow: hidden; flex: 1"
                >
                  <template #default>
                    <span><small>{{ gpuData[gpu.pci].VRAM['Total GTT Usage']?.value || 0 }} / {{ gpuData[gpu.pci].VRAM['Total GTT']?.value || 0 }} MiB</small></span>
                  </template>
                </v-progress-linear>
              </div>
            </v-col>
          </v-row>
          <v-row v-if="gpuData[gpu.pci].gpu_activity" dense class="mt-2">
            <v-col cols="12">
              <div class="text-caption text-medium-emphasis mb-1"><strong>GPU Activity</strong></div>
              <v-row dense>
                <v-col v-if="gpuData[gpu.pci].gpu_activity?.GFX?.value != null" cols="12" md="4">
                  <div style="display: flex; align-items: center; gap: 6px">
                    <span class="text-body-2" style="width: 100px"><small><b>GFX</b></small></span>
                    <v-progress-linear
                      :model-value="gpuData[gpu.pci].gpu_activity.GFX.value || 0"
                      height="12"
                      :color="gpuData[gpu.pci].gpu_activity.GFX.value >= 90 ? 'red' : gpuData[gpu.pci].gpu_activity.GFX.value >= 75 ? 'orange' : 'green'"
                      style="border-radius: 6px; overflow: hidden; flex: 1"
                    >
                      <template #default>
                        <span><small>{{ gpuData[gpu.pci].gpu_activity.GFX.value || 0 }}%</small></span>
                      </template>
                    </v-progress-linear>
                  </div>
                </v-col>
                <v-col v-if="gpuData[gpu.pci].gpu_activity?.MediaEngine?.value != null" cols="12" md="4">
                  <div style="display: flex; align-items: center; gap: 6px">
                    <span class="text-body-2" style="width: 100px"><small><b>Media</b></small></span>
                    <v-progress-linear
                      :model-value="gpuData[gpu.pci].gpu_activity.MediaEngine.value || 0"
                      height="12"
                      :color="gpuData[gpu.pci].gpu_activity.MediaEngine.value >= 90 ? 'red' : gpuData[gpu.pci].gpu_activity.MediaEngine.value >= 75 ? 'orange' : 'green'"
                      style="border-radius: 6px; overflow: hidden; flex: 1"
                    >
                      <template #default>
                        <span><small>{{ gpuData[gpu.pci].gpu_activity.MediaEngine.value || 0 }}%</small></span>
                      </template>
                    </v-progress-linear>
                  </div>
                </v-col>
                <v-col v-if="gpuData[gpu.pci].gpu_activity?.Memory?.value != null" cols="12" md="4">
                  <div style="display: flex; align-items: center; gap: 6px">
                    <span class="text-body-2" style="width: 100px"><small><b>Memory</b></small></span>
                    <v-progress-linear
                      :model-value="gpuData[gpu.pci].gpu_activity.Memory.value || 0"
                      height="12"
                      :color="gpuData[gpu.pci].gpu_activity.Memory.value >= 90 ? 'red' : gpuData[gpu.pci].gpu_activity.Memory.value >= 75 ? 'orange' : 'green'"
                      style="border-radius: 6px; overflow: hidden; flex: 1"
                    >
                      <template #default>
                        <span><small>{{ gpuData[gpu.pci].gpu_activity.Memory.value || 0 }}%</small></span>
                      </template>
                    </v-progress-linear>
                  </div>
                </v-col>
              </v-row>
            </v-col>
          </v-row>
          <v-row v-if="gpuData[gpu.pci].GRBM && hasGrbmData(gpu.pci)" dense class="mt-2">
            <v-col cols="12">
              <details>
                <summary style="cursor: pointer; color: var(--v-theme-primary); text-decoration: underline" class="text-body-2 mb-1">GRBM Engines</summary>
                <v-row dense class="mt-1">
                  <v-col v-for="(engine, engineName) in gpuData[gpu.pci].GRBM" :key="engineName" cols="6" md="4">
                    <div style="display: flex; align-items: center; gap: 6px">
                      <span class="text-caption" style="width: 120px; flex-shrink: 0">{{ engineName }}</span>
                      <v-progress-linear
                        :model-value="engine.value || 0"
                        height="10"
                        :color="engine.value >= 90 ? 'red' : engine.value >= 75 ? 'orange' : 'green'"
                        style="border-radius: 4px; overflow: hidden; flex: 1"
                      />
                      <span class="text-caption" style="width: 40px; text-align: right; flex-shrink: 0">{{ engine.value || 0 }}%</span>
                    </div>
                  </v-col>
                </v-row>
              </details>
            </v-col>
          </v-row>
          <v-row v-if="gpuData[gpu.pci].GRBM2 && hasGrbm2Data(gpu.pci)" dense class="mt-2">
            <v-col cols="12">
              <details>
                <summary style="cursor: pointer; color: var(--v-theme-primary); text-decoration: underline" class="text-body-2 mb-1">GRBM2 Engines</summary>
                <v-row dense class="mt-1">
                  <v-col v-for="(engine, engineName) in gpuData[gpu.pci].GRBM2" :key="engineName" cols="6" md="4">
                    <div style="display: flex; align-items: center; gap: 6px">
                      <span class="text-caption" style="width: 180px; flex-shrink: 0">{{ engineName }}</span>
                      <v-progress-linear
                        :model-value="engine.value || 0"
                        height="10"
                        :color="engine.value >= 90 ? 'red' : engine.value >= 75 ? 'orange' : 'green'"
                        style="border-radius: 4px; overflow: hidden; flex: 1"
                      />
                      <span class="text-caption" style="width: 40px; text-align: right; flex-shrink: 0">{{ engine.value || 0 }}%</span>
                    </div>
                  </v-col>
                </v-row>
              </details>
            </v-col>
          </v-row>
          <v-row v-if="gpuData[gpu.pci].fdinfo && Object.keys(gpuData[gpu.pci].fdinfo).length > 0" dense class="mt-2">
            <v-col cols="12">
              <details>
                <summary style="cursor: pointer; color: var(--v-theme-primary); text-decoration: underline" class="text-body-2 mb-1">Processes</summary>
                <div v-for="(proc, pid) in gpuData[gpu.pci].fdinfo" :key="pid" class="mb-3">
                  <div class="d-flex align-center mb-1">
                    <span class="text-body-2"><b>{{ proc.name || 'Unknown' }}</b></span>
                    <span class="text-caption text-medium-emphasis ml-2">PID: {{ pid }}</span>
                  </div>
                  <v-row v-if="proc.usage?.usage" dense>
                    <v-col v-if="proc.usage.usage.GFX" cols="6" md="3">
                      <div style="display: flex; align-items: center; gap: 6px">
                        <span class="text-caption" style="width: 60px; flex-shrink: 0">GFX</span>
                        <v-progress-linear
                          :model-value="proc.usage.usage.GFX.value || 0"
                          height="10"
                          :color="proc.usage.usage.GFX.value >= 90 ? 'red' : proc.usage.usage.GFX.value >= 75 ? 'orange' : 'green'"
                          style="border-radius: 4px; overflow: hidden; flex: 1"
                        />
                        <span class="text-caption" style="width: 40px; text-align: right; flex-shrink: 0">{{ proc.usage.usage.GFX.value || 0 }}%</span>
                      </div>
                    </v-col>
                    <v-col v-if="proc.usage.usage.Compute" cols="6" md="3">
                      <div style="display: flex; align-items: center; gap: 6px">
                        <span class="text-caption" style="width: 60px; flex-shrink: 0">Compute</span>
                        <v-progress-linear
                          :model-value="proc.usage.usage.Compute.value || 0"
                          height="10"
                          :color="proc.usage.usage.Compute.value >= 90 ? 'red' : proc.usage.usage.Compute.value >= 75 ? 'orange' : 'green'"
                          style="border-radius: 4px; overflow: hidden; flex: 1"
                        />
                        <span class="text-caption" style="width: 40px; text-align: right; flex-shrink: 0">{{ proc.usage.usage.Compute.value || 0 }}%</span>
                      </div>
                    </v-col>
                    <v-col v-if="proc.usage.usage.VRAM" cols="6" md="3">
                      <div style="display: flex; align-items: center; gap: 6px">
                        <span class="text-caption" style="width: 60px; flex-shrink: 0">VRAM</span>
                        <span class="text-caption">{{ proc.usage.usage.VRAM.value || 0 }} {{ proc.usage.usage.VRAM.unit }}</span>
                      </div>
                    </v-col>
                    <v-col v-if="proc.usage.usage.GTT" cols="6" md="3">
                      <div style="display: flex; align-items: center; gap: 6px">
                        <span class="text-caption" style="width: 60px; flex-shrink: 0">GTT</span>
                        <span class="text-caption">{{ proc.usage.usage.GTT.value || 0 }} {{ proc.usage.usage.GTT.unit }}</span>
                      </div>
                    </v-col>
                  </v-row>
                </div>
              </details>
            </v-col>
          </v-row>
        </v-card-text>
        <v-card-text v-else class="text-center pa-8 text-grey">
          Loading data...
        </v-card-text>
      </v-card>
    </div>
    <v-dialog v-model="settingsDialog.value" max-width="600">
      <v-card class="pa-0">
        <v-card-title>Settings</v-card-title>
        <v-card-text>
          <v-form>
            <v-text-field
              v-model.number="settingsDialog.interval"
              label="Update interval (seconds)"
              type="number"
              min="1"
              @blur="validateInterval"
            />
            <v-select
              v-model="settingsDialog.selectedGpus"
              :items="availableGpusItems"
              item-title="title"
              item-value="value"
              label="GPUs"
              multiple
              chips
              clearable
            />
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn color="onPrimary" @click="settingsDialog.value = false">Cancel</v-btn>
          <v-btn color="onPrimary" @click="saveSettings" :loading="settingsDialog.saving">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-fab color="primary" style="position: fixed; bottom: 32px; right: 32px; z-index: 1000" size="large" icon @click="openSettingsDialog">
      <v-icon>mdi-cog</v-icon>
    </v-fab>
  </div>
</template>

<script setup>
import { ref, computed, reactive, onMounted, onUnmounted } from 'vue';

const loading = ref(true);
const settings = ref({ gpus: [], interval: 2 });
const gpuData = ref({});
const availableGpus = ref([]);
let pollInterval = null;

const settingsDialog = reactive({
  value: false,
  interval: 2,
  selectedGpus: [],
  saving: false,
});

const isConfigured = computed(() => {
  return settings.value.gpus && settings.value.gpus.length > 0 && settings.value.gpus.some((g) => g.pci);
});

const availableGpusItems = computed(() => {
  return availableGpus.value.map((gpu) => ({
    title: `${gpu.name} (${gpu.pci})`,
    value: gpu.pci,
  }));
});

const getAuthHeaders = () => ({
  Authorization: 'Bearer ' + localStorage.getItem('authToken'),
});

const getGpuName = (pci) => {
  const gpu = availableGpus.value.find((g) => g.pci === pci);
  return gpu ? gpu.name : `GPU ${pci}`;
};

const validateInterval = () => {
  if (settingsDialog.interval < 1) {
    settingsDialog.interval = 1;
  }
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

const hasGrbmData = (pci) => {
  const grbm = gpuData.value[pci]?.GRBM;
  if (!grbm) return false;
  return Object.values(grbm).some((e) => e.value > 0);
};

const hasGrbm2Data = (pci) => {
  const grbm2 = gpuData.value[pci]?.GRBM2;
  if (!grbm2) return false;
  return Object.values(grbm2).some((e) => e.value > 0);
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
  } catch (e) {
    console.error('Failed to fetch settings:', e);
  }
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
  } catch (e) {
    console.error('Failed to fetch GPUs:', e);
  }
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
  } catch (e) {
    console.error(`Failed to fetch data for GPU ${gpu.pci}:`, e);
  }
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
  const interval = Math.max(1, settings.value.interval) * 1000;
  pollInterval = setInterval(fetchAllGpuData, interval);
};

const stopPolling = () => {
  if (pollInterval) {
    clearInterval(pollInterval);
    pollInterval = null;
  }
};

const openSettingsDialog = () => {
  settingsDialog.interval = settings.value.interval || 2;
  settingsDialog.selectedGpus = settings.value.gpus.map((g) => g.pci).filter(Boolean);
  settingsDialog.saving = false;
  settingsDialog.value = true;
};

const saveSettings = async () => {
  settingsDialog.saving = true;
  validateInterval();

  try {
    const gpus = settingsDialog.selectedGpus.map((pci, idx) => ({ pci, index: idx }));

    const res = await fetch('/api/v1/mos/plugins/settings/amdgpu-top', {
      method: 'POST',
      headers: {
        ...getAuthHeaders(),
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        gpus: gpus,
        interval: settingsDialog.interval,
      }),
    });

    if (res.ok) {
      settings.value = {
        gpus: gpus,
        interval: settingsDialog.interval,
      };
      settingsDialog.value = false;
      startPolling();
    }
  } catch (e) {
    console.error('Failed to save settings:', e);
  } finally {
    settingsDialog.saving = false;
  }
};

onMounted(async () => {
  try {
    await Promise.all([fetchSettings(), fetchAvailableGpus()]);
    if (isConfigured.value) {
      startPolling();
    }
  } catch (e) {
    console.error('Failed to initialize:', e);
  } finally {
    loading.value = false;
  }
});

onUnmounted(() => {
  stopPolling();
});
</script>
