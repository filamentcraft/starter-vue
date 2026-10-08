<script setup lang="ts">
import { Form, Head, Link, router, usePage } from '@inertiajs/vue3';
import {
    ExternalLink,
    Globe,
    House,
    Inbox,
    PanelsTopLeft,
    PencilRuler,
} from '@lucide/vue';
import { computed, ref } from 'vue';
import Heading from '@/components/Heading.vue';
import InputError from '@/components/InputError.vue';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { Spinner } from '@/components/ui/spinner';
import { cn } from '@/lib/utils';
import { inbox, show, status, store } from '@/routes/website';
import type { Team, Website, WebsiteFamily } from '@/types';

type Props = {
    website: Website | null;
    families: WebsiteFamily[];
};

const props = defineProps<Props>();

defineOptions({
    layout: (props: { currentTeam?: Team | null }) => ({
        breadcrumbs: [
            {
                title: 'Website',
                href: props.currentTeam ? show(props.currentTeam.slug) : '/',
            },
        ],
    }),
});

const page = usePage();

const teamSlug = computed(() => page.props.currentTeam?.slug ?? '');

const family = ref(props.families[0]?.value ?? '');
const updating = ref(false);

const homepage = computed(() =>
    props.website?.pages.find((websitePage) => websitePage.isHomepage),
);

const setLive = (live: boolean) => {
    router.visit(status(teamSlug.value), {
        data: { live },
        preserveScroll: true,
        onStart: () => (updating.value = true),
        onFinish: () => (updating.value = false),
    });
};
</script>

<template>
    <Head title="Website" />

    <div class="mx-auto flex w-full max-w-5xl flex-col gap-8 p-4 md:p-6">
        <template v-if="website">
            <header
                class="flex flex-col gap-4 sm:flex-row sm:items-start sm:justify-between"
            >
                <div class="flex flex-col gap-2">
                    <div class="flex items-center gap-2">
                        <h1 class="text-xl font-semibold tracking-tight">
                            {{ website.name }}
                        </h1>
                        <Badge
                            :variant="website.isLive ? 'default' : 'secondary'"
                        >
                            {{ website.statusLabel }}
                        </Badge>
                    </div>
                    <a
                        v-if="website.isLive"
                        :href="website.url"
                        target="_blank"
                        rel="noreferrer"
                        class="inline-flex items-center gap-1 text-sm text-muted-foreground underline-offset-4 hover:underline"
                    >
                        {{ website.url }}
                        <ExternalLink class="size-3.5" />
                    </a>
                    <p v-else class="text-sm text-muted-foreground">
                        Visitors can't see this website until you put it online.
                    </p>
                </div>

                <div class="flex flex-wrap gap-2">
                    <Button
                        variant="outline"
                        :disabled="updating"
                        @click="setLive(!website.isLive)"
                    >
                        <Spinner v-if="updating" />
                        {{ website.isLive ? 'Take offline' : 'Put online' }}
                    </Button>
                    <Button v-if="homepage" as-child>
                        <a :href="homepage.editUrl">
                            <PencilRuler />
                            Open editor
                        </a>
                    </Button>
                </div>
            </header>

            <section class="flex flex-col gap-3">
                <Heading
                    variant="small"
                    title="Pages"
                    description="Add, rename or delete pages from the editor's page menu."
                />

                <ul class="divide-y rounded-xl border">
                    <li
                        v-for="websitePage in website.pages"
                        :key="websitePage.id"
                        class="flex flex-col gap-3 p-4 sm:flex-row sm:items-center sm:justify-between"
                    >
                        <div class="flex min-w-0 flex-col gap-1">
                            <div class="flex items-center gap-2">
                                <House
                                    v-if="websitePage.isHomepage"
                                    class="size-4 text-muted-foreground"
                                />
                                <span class="truncate font-medium">
                                    {{ websitePage.name }}
                                </span>
                                <Badge
                                    v-if="!websitePage.isPublished"
                                    variant="outline"
                                >
                                    Draft
                                </Badge>
                            </div>
                            <span class="text-sm text-muted-foreground">
                                {{ websitePage.path
                                }}{{
                                    websitePage.updatedAt
                                        ? ` · edited ${websitePage.updatedAt}`
                                        : ''
                                }}
                            </span>
                        </div>

                        <div class="flex shrink-0 gap-2">
                            <Button
                                v-if="websitePage.url"
                                variant="ghost"
                                size="sm"
                                as-child
                            >
                                <a
                                    :href="websitePage.url"
                                    target="_blank"
                                    rel="noreferrer"
                                >
                                    <ExternalLink />
                                    View
                                </a>
                            </Button>
                            <Button variant="outline" size="sm" as-child>
                                <a :href="websitePage.editUrl">
                                    <PencilRuler />
                                    Edit
                                </a>
                            </Button>
                        </div>
                    </li>
                </ul>
            </section>

            <section class="grid gap-4 md:grid-cols-2">
                <div class="flex flex-col gap-3 rounded-xl border p-4">
                    <div class="flex items-center gap-2">
                        <PanelsTopLeft class="size-4 text-muted-foreground" />
                        <h2 class="font-medium">Shared areas</h2>
                    </div>
                    <p class="text-sm text-muted-foreground">
                        Shown on every page of the website.
                    </p>
                    <div class="flex flex-wrap gap-2">
                        <template
                            v-for="region in website.regions"
                            :key="region.name"
                        >
                            <Button
                                v-if="region.editUrl"
                                variant="outline"
                                size="sm"
                                as-child
                            >
                                <a :href="region.editUrl">{{ region.name }}</a>
                            </Button>
                        </template>
                    </div>
                </div>

                <div class="flex flex-col gap-3 rounded-xl border p-4">
                    <div class="flex items-center gap-2">
                        <Inbox class="size-4 text-muted-foreground" />
                        <h2 class="font-medium">Inbox</h2>
                        <Badge v-if="website.unreadSubmissions > 0">
                            {{ website.unreadSubmissions }} new
                        </Badge>
                    </div>
                    <p class="text-sm text-muted-foreground">
                        Messages and sign-ups sent from your website's forms.
                    </p>
                    <div>
                        <Button variant="outline" size="sm" as-child>
                            <Link :href="inbox(teamSlug)">Open inbox</Link>
                        </Button>
                    </div>
                </div>
            </section>
        </template>

        <section v-else class="flex flex-col gap-6">
            <Heading
                title="Create your website"
                :description="`Pick a starting style for ${page.props.currentTeam?.name ?? 'your team'}. Every page, color and font stays editable afterwards.`"
            />

            <Form
                v-bind="store.form(teamSlug)"
                class="flex flex-col gap-6"
                v-slot="{ errors, processing }"
            >
                <input type="hidden" name="family" :value="family" />

                <div
                    role="radiogroup"
                    aria-label="Website style"
                    class="grid gap-3 sm:grid-cols-2"
                >
                    <button
                        v-for="option in families"
                        :key="option.value"
                        type="button"
                        role="radio"
                        :aria-checked="family === option.value"
                        :class="
                            cn(
                                'flex flex-col gap-1 rounded-xl border p-4 text-left transition-colors',
                                'hover:bg-accent/50 focus-visible:ring-2 focus-visible:ring-ring focus-visible:outline-none',
                                family === option.value &&
                                    'border-primary bg-accent/40 ring-1 ring-primary',
                            )
                        "
                        @click="family = option.value"
                    >
                        <span class="font-medium">{{ option.label }}</span>
                        <span class="text-sm text-muted-foreground">
                            {{ option.description }}
                        </span>
                    </button>
                </div>

                <InputError :message="errors.family" />

                <div>
                    <Button
                        type="submit"
                        :disabled="processing || family === ''"
                    >
                        <Spinner v-if="processing" />
                        <Globe v-else />
                        Create website
                    </Button>
                </div>
            </Form>
        </section>
    </div>
</template>
