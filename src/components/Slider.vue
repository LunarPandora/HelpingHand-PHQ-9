<script setup>
    import { ref, onMounted, onUnmounted } from 'vue';
    const emit = defineEmits(['sliderUpdate']);
    const props = defineProps({
        sliderValue: Number
    })

    const value = ref(props.sliderValue)
    const min = 0;
    const max = 3;

    const track = ref(null);
    let isDragging = false;

    const updateValueFromPosition = (clientX) => {
        const rect = track.value.getBoundingClientRect();
        let x = clientX - rect.left;
        x = Math.max(0, Math.min(x, rect.width));
        value.value = +(min + (x / rect.width) * (max - min)).toFixed(2);
    };

    const startDrag = (e) => {
        isDragging = true;
        const moveHandler = (ev) => {
            const clientX = ev.touches ? ev.touches[0].clientX : ev.clientX;
            if (isDragging) updateValueFromPosition(clientX);
        };
    
        const endHandler = () => {
            isDragging = false;
            window.removeEventListener('mousemove', moveHandler);
            window.removeEventListener('mouseup', endHandler);
            window.removeEventListener('touchmove', moveHandler);
            window.removeEventListener('touchend', endHandler);

            emit('sliderUpdate', value.value)
        };

        window.addEventListener('mousemove', moveHandler);
        window.addEventListener('mouseup', endHandler);
        window.addEventListener('touchmove', moveHandler);
        window.addEventListener('touchend', endHandler);
    };

    const onTrackClick = (e) => {
        const clientX = e.touches ? e.touches[0].clientX : e.clientX;
        updateValueFromPosition(clientX);
    };
</script>

<template>
    <div class="w-full">
        <div ref="track" @click="onTrackClick" class="w-full bg-gradient-to-r from-[#D6C0B3] via-[#AB886D] to-[#493628] h-[1em] lg:h-[1.5em] rounded-full relative">
            <div
                class="absolute top-1/2 -translate-y-1/2 w-6 lg:w-8 h-6 lg:h-8 bg-white rounded-full cursor-pointer shadow"
                :style="{ left: `${((value - min) / (max - min)) * 100}%`, transform: 'translate(-50%, 0%)' }"
                @mousedown="startDrag"
                @touchstart="startDrag"
            ></div>
        </div>

        <div class="w-full flex items-center justify-between relative mt-2">
            <span class="absolute top-0 left-0 w-0 text-[8px] lg:text-xs text-left">Tidak pernah</span>
            <span class="absolute top-0 left-[30%] text-[8px] lg:text-xs text-center">Beberapa<br> hari</span>
            <span class="absolute top-0 left-[61%] text-[8px] lg:text-xs text-center">Lebih dari<br> separuh waktu<br> yang dimaksud</span>
            <span class="absolute top-0 left-[90%] text-[8px] lg:text-xs text-right">Hampir setiap hari</span>
        </div>
    </div>
</template>