<script setup>
import { ref, onMounted, computed, watch } from 'vue';
import DOMPurify from 'dompurify';

const props = defineProps({
    announcement: {
        type: Object,
        required: true
    }
});

const isVisible = ref(true);

const allowedContentTags = ['b', 'i', 'em', 'strong', 'a', 'p', 'br', 'ul', 'li'];
const allowedContentAttrs = ['href', 'target', 'rel'];

const sanitizedContent = computed(() => {
    const rawContent = props.announcement?.content?.trim()
        ? props.announcement.content
        : '暂无详细内容';
    return DOMPurify.sanitize(rawContent, {
        ALLOWED_TAGS: allowedContentTags,
        ALLOWED_ATTR: allowedContentAttrs
    });
});

const dismiss = (e) => {
    e.stopPropagation(); 
    isVisible.value = false;
    try {
        if (props.announcement.updatedAt) {
            localStorage.setItem(`announcement_dismissed_${props.announcement.updatedAt}`, 'true');
        }
    } catch (e) {
        console.warn('LocalStorage access failed', e);
    }
};

// 立即在 setup 阶段检查，防止 onMounted 导致的闪烁
const checkVisibility = () => {
    if (props.announcement?.dismissible && props.announcement?.updatedAt) {
        const dismissed = localStorage.getItem(`announcement_dismissed_${props.announcement.updatedAt}`);
        if (dismissed) {
            isVisible.value = false;
        } else {
            isVisible.value = true;
        }
    } else {
        isVisible.value = true;
    }
};

checkVisibility();

// 监听公告数据变化（如后台更新了内容），重置可见性
watch(() => props.announcement, () => {
    checkVisibility();
}, { deep: true });

const typeConfig = computed(() => {
    const types = {
        info: {
            color: 'primary',
            gradient: 'from-blue-500/10 to-indigo-500/5',
            border: 'border-blue-200/50 dark:border-blue-800/50',
            text: 'text-blue-700 dark:text-blue-300',
            accent: 'bg-blue-500'
        },
        success: {
            color: 'emerald',
            gradient: 'from-emerald-500/10 to-teal-500/5',
            border: 'border-emerald-200/50 dark:border-emerald-800/50',
            text: 'text-emerald-700 dark:text-emerald-300',
            accent: 'bg-emerald-500'
        },
        warning: {
            color: 'amber',
            gradient: 'from-amber-500/10 to-orange-500/5',
            border: 'border-amber-200/50 dark:border-amber-800/50',
            text: 'text-amber-700 dark:text-amber-300',
            accent: 'bg-amber-500'
        },
        error: {
            color: 'red',
            gradient: 'from-red-500/10 to-rose-500/5',
            border: 'border-red-200/50 dark:border-red-800/50',
            text: 'text-red-700 dark:text-red-300',
            accent: 'bg-red-500'
        }
    };
    return types[props.announcement.type] || types.info;
});

</script>

<template>
    <Transition name="announcement-fade">
        <div v-if="isVisible" 
             class="group relative overflow-hidden transition-all duration-500 misub-radius-lg border backdrop-blur-xl shadow-sm hover:shadow-xl"
             :class="[
                 typeConfig.gradient,
                 typeConfig.border,
                 'bg-white/60 dark:bg-gray-900/40'
             ]">
            
            <!-- Side Accent Line -->
            <div class="absolute left-0 top-0 bottom-0 w-1 opacity-80 transition-all duration-300 group-hover:w-1.5"
                 :class="typeConfig.accent"></div>

            <div class="p-4 sm:p-5">
                <div class="flex items-center gap-4">
                    <!-- Status Icon (Modern Style) -->
                    <div class="flex-shrink-0">
                        <div class="w-10 h-10 rounded-2xl flex items-center justify-center transition-all duration-500 group-hover:scale-110 group-hover:rotate-6 shadow-sm"
                             :class="[
                                 'bg-white/80 dark:bg-white/10',
                                 typeConfig.text
                             ]">
                            <svg v-if="announcement.type === 'info'" xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
                            <svg v-else-if="announcement.type === 'success'" xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
                            <svg v-else-if="announcement.type === 'warning'" xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.268 17c-.77 1.333.192 3 1.732 3z" /></svg>
                            <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
                        </div>
                    </div>

                    <!-- Title Area -->
                    <div class="flex-1 min-w-0 pr-2">
                        <div class="flex flex-col">
                            <h3 class="text-base font-bold text-gray-900 dark:text-white truncate transition-colors duration-300 group-hover:text-primary-600 dark:group-hover:text-primary-400">
                                {{ announcement.title || '系统公告' }}
                            </h3>
                            <span class="text-[10px] font-bold uppercase tracking-widest opacity-40 mt-0.5" :class="typeConfig.text">
                                {{ announcement.type }}
                            </span>
                        </div>
                    </div>

                    <!-- Actions Area -->
                    <div class="flex items-center gap-1.5">
                        <!-- Dismiss Button -->
                        <button v-if="announcement.dismissible" 
                                @click="dismiss"
                                class="p-2 rounded-xl text-gray-400 hover:text-red-500 hover:bg-red-500/10 transition-all duration-300"
                                title="忽略">
                            <svg class="h-4 w-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M6 18L18 6M6 6l12 12" />
                            </svg>
                        </button>
                    </div>
                </div>

                <!-- Content Area (Always Visible) -->
                <div class="mt-5 pt-5 border-t border-gray-200/50 dark:border-white/5">
                    <div class="prose prose-sm dark:prose-invert max-w-none text-gray-600 dark:text-gray-300 leading-relaxed"
                        v-html="sanitizedContent">
                    </div>
                    
                    <div v-if="announcement.updatedAt"
                        class="mt-6 flex items-center justify-between text-[10px] font-bold tracking-wider uppercase opacity-30">
                        <span>MiSub System Update</span>
                        <span>{{ new Date(announcement.updatedAt).toLocaleDateString() }}</span>
                    </div>
                </div>
            </div>

            <!-- Bottom Gradient Glow -->
            <div class="absolute -right-8 -bottom-8 w-32 h-32 blur-3xl rounded-full opacity-10 pointer-events-none transition-transform duration-1000 group-hover:scale-150"
                 :class="typeConfig.accent"></div>
        </div>
    </Transition>
</template>

<style scoped>
.announcement-fade-enter-active,
.announcement-fade-leave-active {
    transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
}

.announcement-fade-enter-from,
.announcement-fade-leave-to {
    opacity: 0;
    transform: translateY(-20px) scale(0.95);
    filter: blur(8px);
}

.animate-announcement-slide {
    animation: announcementSlide 0.4s cubic-bezier(0, 0, 0.2, 1);
}

@keyframes announcementSlide {
    from {
        opacity: 0;
        transform: translateY(-8px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

:deep(.prose a) {
    color: #6366f1;
    text-decoration: none;
    font-weight: 600;
    border-bottom: 2px solid rgba(99, 102, 241, 0.2);
    transition: all 0.2s;
}

:deep(.prose a:hover) {
    border-bottom-color: #6366f1;
}

/* 移动端优化 */
@media (max-width: 640px) {
    .announcement-card {
        margin-left: 0 !important;
        margin-right: 0 !important;
    }
}
</style>
