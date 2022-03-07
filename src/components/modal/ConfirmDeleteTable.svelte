<script>
    /**
     * Imports
     */
    import { fly, fade } from 'svelte/transition';
    import { dashboardLoading, showConfirmDeleteTableModal } from "$stores/stores.js";
    import { X, Warning } from "$icons/svg.js";


    /**
     * Variables
     */
    export let tableName; // Prop variable(s)


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
        $dashboardLoading = true; 

        try {
            // Pass the table name that
            // is needed to be deleted
            await fetch( `http://localhost:8093/api/v1/tables/${ tableName.toLowerCase() }`, {
                method: 'DELETE',
            } );
        } catch ( err ) {
            console.error( err );
        }

        closeModal();
        setTimeout( () => closeModal(), 1500 );
    };
</script>

<section class="modal" out:fade={ { duration: 300 } }>
    <div>
        <section class="modal-ui-table" in:fly={ { y: -200, duration: 300 } }>
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
        </section>
    </div>
</section>