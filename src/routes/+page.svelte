<script>
    import { mailStore } from "$lib/store.js";
    import { version } from "$app/environment"; // 패키지에서 버전 가져옴

    let email = "";
    let description = "";
    let tag = "";
    let showDialog = false; // 데이터 제거 다이얼로그

    // @ts-expect-error
    let targetId = null; // 데이터 제거 시 사용되는 타겟 ID

    let emailError = ""; // 메일 주소 검증 오류 메시지
    const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/; // 텍스트 검증 변수

    function validateEmail() {
        if (!emailRegex.test(email)) {
            emailError = "유효한 이메일 주소를 입력해주세요.";
        } else {
            emailError = "";
        }
    }

    function addMail() {
        validateEmail();
        if (!email || emailError) return;

        mailStore.add({
            id: crypto.randomUUID(),
            email,
            description,
            tag,
            createdAt: new Date().toISOString(),
        });

        email = "";
        description = "";
        tag = "";
    }
</script>

<div class="desktop">
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
                    <strong style="color: red;"
                        >삭제할 경우 다시 복원할 수 없게됩니다.</strong
                    >
                </p>
                <div class="field-row" style="justify-content: flex-end;">
                    <button
                        on:click={() => {
                            // @ts-ignore
                            mailStore.remove(targetId);
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

    <!-- 메인 바디 -->
    <p class="desktop-title">📧 Maile</p>
    <div class="main-flex">
        <!-- 추가 창 -->
        <div class="window" style="width: 260px;">
            <div class="title-bar">
                <div class="title-bar-text">이메일 용도 추가</div>
                <div class="title-bar-controls">
                    <button aria-label="Minimize"></button>
                    <button aria-label="Maximize"></button>
                    <button aria-label="Close"></button>
                </div>
            </div>
            <div class="window-body">
                <div class="field-row-stacked">
                    <label for="text1">이메일 주소</label>
                    <input
                        type="text"
                        id="text1"
                        bind:value={email}
                        on:input={validateEmail}
                        placeholder="ex. 2maile@maile.tld"
                    />
                    {#if emailError}
                        <span class="error-text">{emailError}</span>
                    {/if}
                </div>
                <div class="field-row-stacked">
                    <label for="text2">용도 설명</label>
                    <textarea
                        id="text2"
                        bind:value={description}
                        rows="3"
                        placeholder="ex. 문의 메일만 받는 주소입니다."
                    ></textarea>
                </div>
                <div class="field-row-stacked">
                    <label for="text3">태그</label>
                    <input
                        type="text"
                        id="text3"
                        bind:value={tag}
                        placeholder="비즈니스"
                    />
                </div>
                <div
                    class="field-row"
                    style="margin-top: 10px; justify-content: flex-end;"
                >
                    <button class="default" on:click={addMail}>생성</button>
                </div>
            </div>
        </div>

        <!-- 리스트 창 -->
        <div class="window" style="width: 380px;">
            <div class="title-bar">
                <div class="title-bar-text">용도 리스트</div>
                <div class="title-bar-controls">
                    <button aria-label="Minimize"></button>
                    <button aria-label="Maximize"></button>
                    <button aria-label="Close"></button>
                </div>
            </div>
            <div class="window-body" style="padding: 0;">
                {#if $mailStore.length === 0}
                    <p style="padding: 8px;">등록된 메일이 없습니다.</p>
                {:else}
                    <ul class="mail-list tree-view">
                        {#each $mailStore as mail}
                            <li class="mail-item">
                                <a href="/mail/{mail.id}" class="mail-link">
                                    <strong>{mail.email}</strong>
                                    {#if mail.tag}
                                        <span class="tag-badge"
                                            >[{mail.tag}]</span
                                        >
                                    {/if}
                                    {#if mail.description}
                                        <p class="mail-desc">
                                            {mail.description}
                                        </p>
                                    {/if}
                                </a>
                                <button
                                    on:click={() => {
                                        targetId = mail.id;
                                        showDialog = true;
                                    }}>삭제</button
                                >
                            </li>
                        {/each}
                    </ul>
                {/if}
            </div>
            <div class="status-bar">
                <p class="status-bar-field">총 {$mailStore.length}개</p>
                <p class="status-bar-field">v{version}</p>
            </div>
        </div>
    </div>
</div>
