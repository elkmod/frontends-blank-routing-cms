<script setup lang="ts">

import { SeoUrl } from "@shopware-pwa/types"

import { useNavigation, useNavigationContext, useNavigationSearch, useCategorySearch } from '@shopware-pwa/composables-next';
import { useContext } from "unctx/index";

const { loadNavigationElements, navigationElements } = useNavigation();
await loadNavigationElements({ depth: 2 });

const { resolvePath } = useNavigationSearch();
const route = useRoute()

const seoResult: SeoUrl|null = await resolvePath(route.path);

const { routeName, foreignKey } = useNavigationContext(ref(seoResult));

const data = ref(null);

const fetchEntity = async () => {
    switch(routeName.value) {
        case 'frontend.navigation.page':
            let { search: categorySearch } = useCategorySearch();
            const categoryResponse = await categorySearch(foreignKey.value, {
                withCmsAssociations: true
            });
            const { category } = useCategory(categoryResponse);
            data.value = category;
            break;
        case 'frontend.detail.page':
            let { search: productSearch } = useProductSearch();
            const productResponse = await productSearch(foreignKey.value);
            const { product } = useProduct(productResponse);
            data.value = product;
            break;
        case 'frontend.landing.page':
            let { search: landingSearch } = useLandingSearch();
            const landing = await landingSearch(foreignKey.value);
            data.value = landing;
            break;
        default:
            return null;
    }
};

watch(foreignKey, fetchEntity);

onMounted(fetchEntity);
</script>

<template>
    <ul>
        <li
         v-for="navigationElement in navigationElements"
         :key="navigationElement.id"
        >
            <router-link :to="'/' + navigationElement.seoUrls[0]?.seoPathInfo">
                {{ navigationElement.translated.name }}
            </router-link>
        </li>
    </ul>
    <hr>
    
    <CmsPage v-if="data" :content="data.value.cmsPage"/>
</template>
