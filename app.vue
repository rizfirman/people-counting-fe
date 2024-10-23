<template>
  <div>
    <h1>Live CCTV Streams</h1>
    <div v-for="(cctv, index) in cctvData" :key="index" class="video-container">
      <h2>CCTV Stream {{ cctv.id }}</h2>
      <p>Total Count: {{ cctv.totalCount }}</p>
      <img :src="`http://localhost:9000${cctv.link}?width=640&height=480`" alt="CCTV Stream" />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import io from 'socket.io-client';

const cctvData = ref([]);

// Fetch initial CCTV links
const fetchCCTVLinks = async () => {
  const res = await $fetch('http://localhost:9000/cctv_links');
  cctvData.value = res.data;
};

onMounted(() => {
  fetchCCTVLinks();

  const socket = io('http://localhost:9000'); // Connect to WebSocket server

  // Listen for 'update_count' events
  socket.on('update_count', (data) => {
    const streamId = data.stream_id;
    const updatedCount = data.totalCount;

    // Update totalCount for the corresponding stream
    const stream = cctvData.value.find((cctv) => cctv.id === streamId);
    if (stream) {
      stream.totalCount = updatedCount;
    }
  });
});
</script>

<style scoped>
.video-container {
  margin-bottom: 20px;
}
</style>
