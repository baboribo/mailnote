<script>
    import { mailStore } from "$lib/store.js";

    let email = "";
    let description = "";
    let tag = "";

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

<main>
    <h1>Mail</h1>
    <div>
        <input
            type="text"
            bind:value={email}
            on:input={validateEmail}
            placeholder="메일 주소"
        />
        {#if emailError}
            <span style="color: red">{emailError}</span>
        {/if}
        <input bind:value={description} placeholder="설명" />
        <input bind:value={tag} placeholder="태그" />
        <button on:click={addMail}>추가</button>
    </div>
    <ul>
        {#each $mailStore as mail}
            <li>
                <a href="/mail/{mail.id}">
                    <strong>{mail.email}</strong>
                    <span>{mail.tag}</span>
                    <p>{mail.description}</p>
                </a>
                <button
                    on:click={() => {
                        mailStore.remove(mail.id);
                    }}>삭제</button
                >
            </li>
        {:else}
            <p>등록된 메일이 없습니다.</p>
        {/each}
    </ul>
</main>
