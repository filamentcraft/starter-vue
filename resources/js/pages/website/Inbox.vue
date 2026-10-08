<script setup lang="ts">
import { Head, router, usePage } from '@inertiajs/vue3';
import { Check, Inbox as InboxIcon } from '@lucide/vue';
import Heading from '@/components/Heading.vue';
import { Badge } from '@/components/ui/badge';
import { Button } from '@/components/ui/button';
import { cn } from '@/lib/utils';
import { inbox, show } from '@/routes/website';
import { read } from '@/routes/website/inbox';
import type { Team, WebsiteSubmission } from '@/types';

type Props = {
    submissions: WebsiteSubmission[];
};

defineProps<Props>();

defineOptions({
    layout: (props: { currentTeam?: Team | null }) => ({
        breadcrumbs: [
            {
                title: 'Website',
                href: props.currentTeam ? show(props.currentTeam.slug) : '/',
            },
            {
                title: 'Inbox',
                href: props.currentTeam ? inbox(props.currentTeam.slug) : '/',
            },
        ],
    }),
});

const page = usePage();

const markAsRead = (submission: WebsiteSubmission) => {
    router.visit(
        read({
            current_team: page.props.currentTeam?.slug ?? '',
            submission: submission.id,
        }),
        { preserveScroll: true },
    );
};
</script>

<template>
    <Head title="Inbox" />

    <div class="mx-auto flex w-full max-w-5xl flex-col gap-6 p-4 md:p-6">
        <Heading
            title="Inbox"
            description="Messages and sign-ups sent from your website's contact and newsletter forms."
        />

        <div
            v-if="submissions.length === 0"
            class="flex flex-col items-center gap-2 rounded-xl border border-dashed p-10 text-center"
        >
            <InboxIcon class="size-6 text-muted-foreground" />
            <p class="font-medium">Nothing here yet</p>
            <p class="max-w-sm text-sm text-muted-foreground">
                Add a Contact or Newsletter section in the editor and new
                messages will show up here.
            </p>
        </div>

        <ul v-else class="divide-y rounded-xl border">
            <li
                v-for="submission in submissions"
                :key="submission.id"
                :class="
                    cn(
                        'flex flex-col gap-3 p-4',
                        !submission.isRead && 'bg-accent/30',
                    )
                "
            >
                <div class="flex flex-wrap items-center justify-between gap-2">
                    <div class="flex items-center gap-2">
                        <span class="font-medium">
                            {{ submission.sender || 'Anonymous visitor' }}
                        </span>
                        <Badge variant="secondary">
                            {{ submission.typeLabel }}
                        </Badge>
                        <Badge v-if="!submission.isRead">New</Badge>
                    </div>
                    <div class="flex items-center gap-3">
                        <span class="text-sm text-muted-foreground">
                            {{ submission.receivedAt }}
                        </span>
                        <Button
                            v-if="!submission.isRead"
                            variant="ghost"
                            size="sm"
                            @click="markAsRead(submission)"
                        >
                            <Check />
                            Mark as read
                        </Button>
                    </div>
                </div>

                <dl
                    class="grid gap-x-6 gap-y-1 text-sm sm:grid-cols-[max-content_1fr]"
                >
                    <div
                        v-for="(value, field) in submission.fields"
                        :key="field"
                        class="contents"
                    >
                        <dt class="text-muted-foreground capitalize">
                            {{ String(field).replaceAll('_', ' ') }}
                        </dt>
                        <dd class="break-words whitespace-pre-line">
                            {{ value }}
                        </dd>
                    </div>
                </dl>
            </li>
        </ul>
    </div>
</template>
