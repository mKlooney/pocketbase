<script>
    import { slide } from "svelte/transition";
    import { Collection, Record } from "pocketbase";
    import CommonHelper from "@/utils/CommonHelper";
    import tooltip from "@/actions/tooltip";
    import { confirm } from "@/stores/confirmation";
    import { removeError } from "@/stores/errors";
    import Field from "@/components/base/Field.svelte";

    export let collection = new Collection();
    export let record = new Record();

    let originalUsername = record.username || null;

    let changePasswordToggle = false;

    $: if (!record.username && record.username !== null) {
        record.username = null;
    }

    $: if (!changePasswordToggle) {
        record.password = null;
        record.passwordConfirm = null;
        removeError("password");
        removeError("passwordConfirm");
    }
</script>

<div class="grid m-b-base">
    <div class="col-lg-6">
        <Field class="form-field {!record.isNew ? 'required' : ''}" name="username" let:uniqueId>
            <label for={uniqueId}>
                <i class={CommonHelper.getFieldTypeIcon("user")} />
                <span class="txt">Username</span>
            </label>
            <input
                type="text"
                requried={!record.isNew}
                placeholder={record.isNew ? "Leave empty to auto generate..." : originalUsername}
                id={uniqueId}
                bind:value={record.username}
            />
        </Field>
    </div>

    <div class="col-lg-6">
        <Field
            class="form-field {collection.options?.requireEmail ? 'required' : ''}"
            name="email"
            let:uniqueId
        >
            <label for={uniqueId}>
                <i class={CommonHelper.getFieldTypeIcon("email")} />
                <span class="txt">Email</span>
            </label>
            <div class="form-field-addon email-visibility-addon">
                <button
                    type="button"
                    class="btn btn-sm btn-transparent {record.emailVisibility ? 'btn-success' : 'btn-hint'}"
                    use:tooltip={{
                        text: "Make email public or private",
                        position: "top-right",
                    }}
                    on:click={() => (record.emailVisibility = !record.emailVisibility)}
                >
                    <span class="txt">Public: {record.emailVisibility ? "On" : "Off"}</span>
                </button>
            </div>
            <!-- svelte-ignore a11y-autofocus -->
            <input
                type="email"
                autofocus={record.isNew}
                autocomplete="off"
                id={uniqueId}
                required={collection.options?.requireEmail}
                bind:value={record.email}
            />
        </Field>
    </div>

    <div class="col-lg-12">
        {#if !record.isNew}
            <Field class="form-field form-field-toggle" name="verified" let:uniqueId>
                <input type="checkbox" id={uniqueId} bind:checked={changePasswordToggle} />
                <label for={uniqueId}>Change password</label>
            </Field>
        {/if}

        {#if record.isNew || changePasswordToggle}
            <div class="block" transition:slide|local={{ duration: 150 }}>
                <div class="grid" class:p-t-xs={changePasswordToggle}>
                    <div class="col-sm-6">
                        <Field class="form-field required" name="password" let:uniqueId>
                            <label for={uniqueId}>
                                <i class="ri-lock-line" />
                                <span class="txt">Password</span>
                            </label>
                            <input
                                type="password"
                                autocomplete="new-password"
                                id={uniqueId}
                                required
                                bind:value={record.password}
                            />
                        </Field>
                    </div>
                    <div class="col-sm-6">
                        <Field class="form-field required" name="passwordConfirm" let:uniqueId>
                            <label for={uniqueId}>
                                <i class="ri-lock-line" />
                                <span class="txt">Password confirm</span>
                            </label>
                            <input
                                type="password"
                                autocomplete="new-password"
                                id={uniqueId}
                                required
                                bind:value={record.passwordConfirm}
                            />
                        </Field>
                    </div>
                </div>
            </div>
        {/if}
    </div>

    <div class="col-lg-12">
        <Field class="form-field form-field-toggle" name="verified" let:uniqueId>
            <input
                type="checkbox"
                id={uniqueId}
                bind:checked={record.verified}
                on:change|preventDefault={(e) => {
                    if (record.isNew) {
                        return; // no confirmation required
                    }
                    confirm(
                        `Do you really want to manually change the verified account state?`,
                        () => {},
                        () => {
                            record.verified = !e.target.checked;
                        }
                    );
                }}
            />
            <label for={uniqueId}>Verified</label>
        </Field>
    </div>
</div>

<style>
    .email-visibility-addon ~ input {
        padding-right: 100px;
    }
</style>
