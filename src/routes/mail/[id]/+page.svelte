<script>
    // URL에서 "https://"가 빠진 경우 자동으로 추가하는 기능 만들기 - 현재는 href에서 https 붙여서 열도록 일시적으로 다듬었음.

    import { page } from "$app/stores";
    import { mailStore } from "$lib/store.js";
    import { derived } from "svelte/store";

    let serviceName = "";
    let serviceUrl = "";

    let showDialog = false; // 데이터 제거 다이얼로그

    // @ts-expect-error
    let targetId = null; // 데이터 제거 시 사용되는 타겟 ID

    const id = /** @type {string} */ ($page.params.id);

    const mail = derived(mailStore, ($mailStore) =>
        $mailStore.find(
            /** @type {function(any): boolean} */ (m) => m.id === id,
        ),
    );

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

<!-- 이메일 용도 삭제 다이얼로그 -->
{#if showDialog}
    <div
        class="window"
        style="width: 200px; position: fixed; top: 42%; left: 45%;"
    >
        <div class="title-bar">
            <div class="title-bar-text">용도 주소 삭제</div>
            <div class="title-bar-controls">
                <button
                    aria-label="Close"
                    on:click={() => {
                        targetId = null;
                        showDialog = false;
                    }}
                ></button>
            </div>
        </div>
        <div class="window-body">
            <p>
                정말로 삭제하시겠습니까? <br />
                <span style="color: red;"
                    >삭제할 경우 다시 복원할 수 없게됩니다.</span
                >
            </p>
            <div class="field-row" style="justify-content: flex-end;">
                <button
                    on:click={() => {
                        // @ts-ignore
                        mailStore.removeService(id, targetId);
                        showDialog = false;
                    }}>예</button
                >
                <button
                    on:click={() => {
                        targetId = null;
                        showDialog = false;
                    }}>아니오</button
                >
            </div>
        </div>
    </div>
{/if}

{#if $mail}
    <a href="/">목록으로 돌아가기</a>
    <h1 class="desktop-title-email">{$mail.email}</h1>
    <p>{$mail.description}</p>
    <span>{$mail.tag}</span>

    <h2 class="desktop-title">서비스 목록</h2>
    <div>
        <input bind:value={serviceName} placeholder="서비스 이름" />
        <input bind:value={serviceUrl} placeholder="URL (선택)" />
        <button on:click={addService}>서비스 추가</button>
    </div>

    <ul>
        {#each $mail.services || [] as service}
            <li>
                {#if service.url}
                    <a href="https://{service.url}" target="_blank"
                        >{service.name}</a
                    >
                {:else}
                    <span>{service.name}</span>
                {/if}
                <button
                    on:click={() => {
                        targetId = service.id;
                        showDialog = true;
                    }}>삭제</button
                >
            </li>
        {/each}
    </ul>
{:else}
    <p>서비스를 불러올 수 없습니다.</p>
{/if}
