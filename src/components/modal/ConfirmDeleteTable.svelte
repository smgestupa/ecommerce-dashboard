<script>
    /**
     * Imports
     */
    import { fly, fade } from 'svelte/transition';
    import { showConfirmDeleteTableModal } from "$stores/stores.js";
    import { X, Warning } from "$icons/svg.js";
    import ModalLoading from "$components/modal/components/ModalLoading.svelte";
    import ModalStatus from "$components/modal/components/ModalStatus.svelte";


    /**
     * Variables
     */
    export let tableName; // Prop variable(s)
    export let fetchTables; // Prop function(s)
    const statusMessage = "You have successfully deleted the table " + tableName;
    let modalLoading = false, statusCode = undefined;


    /**
     * Will close this
     * component modal
     */
    const closeModal = () => $showConfirmDeleteTableModal = false;

    /**
     * Will be used to delete
     * a table
     */
    const deleteTable = async () => {
        modalLoading = true; 

        try {
            // Pass the table name that
            // is needed to be deleted
            const req = await fetch( `http://localhost:8093/api/v1/tables/${ tableName.toLowerCase() }`, {
                method: 'DELETE',
            } );

            statusCode = req.status;
            fetchTables();
        } catch ( err ) {
            console.error( err );
        }

        modalLoading = false;
        setTimeout( () => closeModal(), 1500 );
    };
</script>

<section class="modal" out:fade={ { duration: 300 } }>
    <div>
        <section class="modal-ui-table" in:fly={ { y: -200, duration: 300 } }>
            { #if modalLoading }
                <ModalLoading />
            { :else if statusCode !== undefined }
                <ModalStatus statusCode={ statusCode }
                statusMessage={ statusMessage }/>
            { :else }
                <header class="modal-header">
                    <!-- Modal notif title -->
                    <h1>You are about to delete the <span class="modal-header-delete">{ tableName }</span> table</h1>

                    <!-- Modal close button -->
                    <section class="modal-close">
                        <button on:click={ () => closeModal() }>
                            <X />
                        </button>
                    </section>
                </header>

                <!-- Modal notif description -->
                <section class="modal-description-warning">
                    <figure>
                        <Warning class={ "warning-modal-icon" } />
                    </figure>
                    <h3>Please confirm the action that you intend to do.</h3>
                </section>

                <!-- Modal buttons -->
                <section class="modal-buttons">
                    <!-- Confirm button -->
                    <div class="modal-button-delete"
                    on:click={ () => deleteTable() }>
                        <button>Confirm</button>
                    </div>

                    <!-- Cancel button -->
                    <div class="modal-button-cancel"
                    on:click={ () => closeModal() }>
                        <button>Cancel</button>
                    </div>
                </section>
            { /if }
        </section>
    </div>
</section>