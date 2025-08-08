<script lang="ts">
    import { t } from "$lib/i18n/translations";
    import cachedInfo from "$lib/state/server-info";
    import { getServerInfo } from "$lib/api/server-info";

    import Skeleton from "$components/misc/Skeleton.svelte";
    import IconPlus from "@tabler/icons-svelte/IconPlus.svelte";
    import PopoverContainer from "$components/misc/PopoverContainer.svelte";

    let services: string[] = [];
    let statusMap: Record<string, 'available' | 'issue' | 'unavailable'> = {};

    $: expanded = false;
    let servicesContainer: HTMLDivElement;
    $: loaded = false;

    const loadInfo = async () => {
        await getServerInfo();

        if ($cachedInfo) {
            services = $cachedInfo.info.cobalt.services;

            try {
                const res = await fetch('/api/services/status.js');
                if (res.ok) {
                    statusMap = await res.json();
                } else {
                    console.error("Error fetching services status");
                }
            } catch (err) {
                console.error("Unable to retrieve service status", err);
            }

            loaded = true;
        }
    };

    const popoverAction = async () => {
        expanded = !expanded;
        if (expanded && services.length === 0) {
            await loadInfo();
        }
        if (expanded) {
            servicesContainer.focus();
        }
    };

    const statusLabel = {
        available: "Available",
        issue: "Has issue",
        unavailable: "Unavailable"
    };
</script>

<div id="supported-services" class:expanded>
    <button
        id="services-button"
        class="button"
        on:click={popoverAction}
        aria-label={$t(`save.services.title_${expanded ? "hide" : "show"}`)}
    >
        <div class="expand-icon">
            <IconPlus />
        </div>
        <span class="title">{$t("save.services.title")}</span>
    </button>

    <PopoverContainer id="services-popover" {expanded}>
        <div
            id="services-container"
            bind:this={servicesContainer}
            tabindex="-1"
        >
            {#if loaded}
                {#each services as service}
                    <div
                        class="service-item {statusMap[service] || 'available'}"
                        title={statusLabel[statusMap[service] || 'available']}
                    >
                        {service}
                    </div>
                {/each}
            {:else}
                {#each { length: 17 } as _}
                    <Skeleton
                        class="elevated"
                        width={Math.random() * 44 + 50 + "px"}
                        height="24.5px"
                    />
                {/each}
            {/if}
        </div>
        <div id="services-disclaimer" class="subtext">
            {$t("save.services.disclaimer")}
        </div>
    </PopoverContainer>
</div>

<style>
    .service-item {
        display: flex;
        padding: 4px 8px;
        border-radius: calc(var(--border-radius) / 2);
        font-size: 12.5px;
        font-weight: 500;
        cursor: default;
    }

    .available {
        background: #e6f4ea;
        color: #137333;
    }

    .issue {
        background: #fff4e5;
        color: #a15c00;
    }

    .unavailable {
        background: #fce8e6;
        color: #a50e0e;
    }

    .service-item:hover.available {
        background: #d6f0de;
    }

    .service-item:hover.issue {
        background: #ffe9cc;
    }

    .service-item:hover.unavailable {
        background: #f8d7d5;
    }
</style>
