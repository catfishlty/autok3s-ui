<template>
  <div v-if="routeGroup.children && routeGroup.children.length > 0" :key="routeGroup.name">
    <div class="grid grid-cols-[1fr,22px] items-center py-5px cursor-pointer hover:bg-gray-200" @click="toggleOpen(routeGroup.name)">
      <h6 class="pl-8px">{{routeGroup.title}}</h6>
      <k-icon type="arrow-right" :direction="expandedStateMap[routeGroup.name] ? 'up':''" class="justify-self-center"></k-icon>
    </div>
    <ul :class="[expandedStateMap[routeGroup.name] ? 'block': 'hidden']">
      <li v-for="child in routeGroup.children" :key="child.name">
        <nav-group :route-group="child" :expanded-state-map="expandedStateMap"></nav-group>
      </li>
    </ul>
  </div>
  <!-- <router-link v-else
    class="k-nav-group-link"
    :class="{'grid-cols-[20px,1fr]': routeGroup.level > 1}"
    :to="{name: routeGroup.name}">
    <k-icon v-if="routeGroup.level > 1" :type="routeGroup.icon || 'folder'" class="k-nav-group-link__icon"></k-icon>
    <span>{{routeGroup.title}}</span>
  </router-link> -->
  <router-link v-else
    :to="{name: routeGroup.name}"
    custom
    v-slot="{ navigate, href, route, isActive, isExactActive}">
    <a
      :href="href"
      @click="navigate"
      class="k-nav-group__link grid grid-cols-[auto,1fr] gap-2px items-center py-3px"
      :class="[isActive || $route.path.startsWith(route.path) ? 'router-link-active bg-gray-100' : 'hover:bg-gray-200', isExactActive ? 'router-link-exact-active' : '', routeGroup.level > 1 ? 'grid-cols-[20px,1fr] pl-16px': 'pl-8px']">
      <k-icon class="k-nav-group__icon" v-if="routeGroup.level > 1" :type="routeGroup.icon || 'folder'" :class="[isActive || $route.path.startsWith(route.path) ? 'k-nav-group__icon--active': '']"></k-icon>
      <span class="overflow-hidden overflow-ellipsis whitespace-nowrap">{{routeGroup.title}}</span>
    </a>
  </router-link>
</template>
<script>
import KIcon from '@/components/Icon'
export default {
  props: {
    routeGroup: {
      type: Object,
      required: true,
    },
    expandedStateMap: {
      type: Object,
      required: true
    },
  },
  emits: ['toggle-open'],
  name: 'NavGroup',
  setup(props, context) {
    const {emit} = context
    const toggleOpen = (name) => {
      emit('toggle-open', name)
    }
    return {
      toggleOpen
    }
  },
  components: {
    KIcon
  }
}
</script>
<style>
.k-nav-group__link:hover .k-nav-group__icon {
  @apply text-gray-800;
}
.k-nav-group__icon {
  @apply text-gray-400;
}
.k-nav-group__icon--active {
  @apply text-gray-800;
}
</style>
