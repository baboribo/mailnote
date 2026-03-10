<script>
    import { page } from "$app/stores";
    import { mailStore } from "$lib/store.js";
    import { derived } from "svelte/store";

    const id = /** @type {string} */ ($page.params.id);

    const mail = derived(mailStore, ($mailStore) =>
        $mailStore.find(
            /** @type {function(any): boolean} */ (m) => m.id === id,
        ),
    );

    let serviceName = "";
    let serviceUrl = "";

    function addService() {
        if (!serviceName) return;

        mailStore.addService(id, {
            id: crypto.randomUUID(),
            name: serviceName,
            url: serviceUrl,
        });

        serviceName = "";
        serviceUrl = "";
    }
</script>

{#if $mail}
    <a href="/">목록으로 돌아가기</a>
    <h1>{$mail.email}</h1>
    <p>{$mail.description}</p>
    <span>{$mail.tag}</span>

    <h2>서비스 목록</h2>
    <div>
        <input bind:value={serviceName} placeholder="서비스 이름" />
        <input bind:value={serviceUrl} placeholder="URL (선택)" />
        <button on:click={addService}>서비스 추가</button>
    </div>

    <ul>
        {#each $mail.services || [] as service}
            <li>
                {#if service.url}
                    <a href={service.url} target="_blank">{service.name}</a>
                {:else}
                    <span>{service.name}</span>
                {/if}
                <button
                    on:click={() => {
                        mailStore.removeService(id, service.id);
                    }}>삭제</button
                >
            </li>
        {/each}
    </ul>
{:else}
    <p>서비스를 불러올 수 없습니다.</p>
{/if}
