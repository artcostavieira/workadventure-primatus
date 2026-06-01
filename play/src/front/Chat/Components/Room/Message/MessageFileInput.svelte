<script lang="ts">
    import { get } from "svelte/store";
    import { createEventDispatcher, onDestroy, onMount, tick } from "svelte";
    import { selectedChatMessageToReply } from "../../../Stores/ChatStore";
    import { chatInputFocusStore } from "../../../../Stores/ChatStore";
    import { IconLoader, IconPaperclip, IconX } from "@wa-icons";
    import LL from "../../../../../i18n/i18n-svelte";

    const dispatch = createEventDispatcher<{
        filesSelected: FileList;
        fileUploaded: void;
    }>();

    let files: FileList | undefined = undefined;
    let fileInputElement: HTMLInputElement;
    let pickerOpening = false;
    let pickerOpeningTimeout: ReturnType<typeof setTimeout> | undefined = undefined;

    $: {
        if (files && files.length > 0) {
            stopPickerOpening();
            dispatch("filesSelected", files);
            dispatch("fileUploaded");
            files = undefined;
            fileInputElement.value = "";
        }
    }

    function unselectChatMessageToReplyIfSelected() {
        if (get(selectedChatMessageToReply) !== null) {
            selectedChatMessageToReply.set(null);
        }
        dispatch("fileUploaded");
    }

    function stopPickerOpening() {
        pickerOpening = false;
        if (pickerOpeningTimeout) {
            clearTimeout(pickerOpeningTimeout);
            pickerOpeningTimeout = undefined;
        }
    }

    async function openFilePicker() {
        if (!fileInputElement) {
            return;
        }

        pickerOpening = true;
        await tick();
        fileInputElement.click();

        if (pickerOpeningTimeout) {
            clearTimeout(pickerOpeningTimeout);
        }
        pickerOpeningTimeout = setTimeout(() => {
            pickerOpening = false;
            pickerOpeningTimeout = undefined;
        }, 1_000);
    }

    function focusChatInput() {
        // Disable input manager to prevent the game from receiving the input
        chatInputFocusStore.set(true);
    }
    function unfocusChatInput() {
        // Enable input manager to allow the game to receive the input
        chatInputFocusStore.set(false);
    }

    onMount(() => {
        window.addEventListener("focus", stopPickerOpening);
        openFilePicker().catch((error) => console.error(error));
    });

    onDestroy(() => {
        stopPickerOpening();
        window.removeEventListener("focus", stopPickerOpening);
    });
</script>

<div class="relative">
    <input
        id="upload"
        class="hidden"
        type="file"
        multiple
        bind:files
        bind:this={fileInputElement}
        data-testid="uploadChatCustomAsset"
        on:focusin={focusChatInput}
        on:focusout={unfocusChatInput}
        on:change={stopPickerOpening}
    />
    <button
        type="button"
        class="p-0 m-0 h-11 w-11 flex items-center justify-center hover:bg-white/10 rounded-none disabled:opacity-50"
        aria-label={$LL.chat.fileAttachment.title()}
        on:click={openFilePicker}
        disabled={pickerOpening}
    >
        {#if pickerOpening}
            <IconLoader class="animate-spin" font-size={18} />
        {:else}
            <IconPaperclip class="hover:!cursor-pointer" font-size={18} />
        {/if}
    </button>
    <button
        class="absolute top-0 right-0 m-1 hover:bg-white/10 cursor-pointer"
        on:click={() => unselectChatMessageToReplyIfSelected()}
    >
        <IconX class=" text-white/50 hover:text-white transition-all" font-size={16} />
    </button>
</div>
