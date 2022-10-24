<script>
  import "../app.css";
  import { onMount } from "svelte";
  import Fa from "svelte-fa/src/fa.svelte";
  import {
    faFlag,
    faClose,
    faRotateLeft,
    faRotateRight,
  } from "@fortawesome/free-solid-svg-icons";

  let cropper;
  let image;
  let URL;
  let modal;
  let inputImage;
  let blobArry;
  const filesPath = "/";
  const fileExtention = ".webp";

  onMount(() => {
    let container = document.querySelector(".img-container");
    image = container.getElementsByTagName("img").item(0);
    URL = window.URL || window.webkitURL;
    modal = document.querySelector("#image-edit-modal");
    inputImage = document.querySelector("[type='file']");
    blobArry = [];
  });

  function insertFile() {
    modal = document.querySelector("#image-edit-modal");
    modal.classList.remove("hidden");

    let files = this.files;
    let file;
    if (files && files.length) {
      file = files[0];
      if (/^image\/\w+/.test(file.type)) {
        image.src = URL.createObjectURL(file);

        if (cropper) {
          cropper.destroy();
        }

        // @ts-ignore
        cropper = new Cropper(image, {});
        inputImage.value = null;
      } else {
        window.alert("Please choose an image file.");
      }
    }
  }

  function closeModal() {
    modal.classList.add("hidden");
    inputImage.value = null;
  }

  function cropImage() {
    const id = Date.now();
    cropper.getCroppedCanvas().toBlob(
      (blob) => {
        blob.id = id;
        blobArry.push(blob);
      } /*, 'image/png' */
    );

    const croppedImage = cropper.getCroppedCanvas();
    let ul = document.querySelector(".image-list ul");
    let li = document.createElement("li");
    let div = document.createElement("div");

    const closeIcon = `<span data-id="${id}" class="bg-gray-200 hover:bg-gray-300 cursor-pointer rounded-full w-6 h-6 flex justify-center items-center close-icon color-white absolute top-0 right-4">
				<svg 
					class="close-icon svelte-fa s-z-WEjw8Gh1FG" viewBox="0 0 320 512" aria-hidden="true" 
					role="img" xmlns="http://www.w3.org/2000/svg" 
					style="height: 1em; vertical-align: -0.125em; 
					transform-origin: center center; overflow: visible;">
					<g transform="translate(160 256)" class="s-z-WEjw8Gh1FG close-icon">
						<g transform="translate(0,0) scale(1,1)" class="close-icon s-z-WEjw8Gh1FG">
							<path d="M310.6 150.6c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0L160 210.7 54.6 105.4c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3L114.7 256 9.4 361.4c-12.5 12.5-12.5 32.8 0 45.3s32.8 12.5 45.3 0L160 301.3 265.4 406.6c12.5 12.5 32.8 12.5 45.3 0s12.5-32.8 0-45.3L205.3 256 310.6 150.6z"
							fill="currentColor" transform="translate(-160 -256)" class="close-icon s-z-WEjw8Gh1FG">
							</path>
						</g>
					</g>
				</svg>
			</span>`;

    div.innerHTML = closeIcon;

    croppedImage.classList.add("max-w-full");

    li.classList.add("mt-4", "relative");
    li.append(croppedImage);
    li.append(div);
    ul.append(li);
    closeModal();
    remove();
  }

  function remove() {
    document.addEventListener("click", function (e) {
      if (e.target.classList.contains("close-icon")) {
        const id =
          e.target.closest("span").getAttribute("data-id") ||
          e.target.getAttribute("data-id");
        const li = e.target.closest("li");
        li ? li.remove() : "";

        blobArry = blobArry.filter((b) => b.id != id);
      }
    });
  }

  async function uploadImages() {
    let formData = new FormData();
    /* Other parts */
    formData.append("file", blobArry[0]);
    formData.append("name", `${blobArry[0].id + fileExtention}`);
    formData.append("path", filesPath);
    /* Calls the process on the server */
    await fetch(`/upload`, {
      method: "POST",
      body: formData,
    })
      .then((res) => {
			console.log(res);
        if (!res.ok) {
          // the response has a bad status (500..)
          throw new Error(
            "upload error status " +
              res.status +
              ", status text: " +
              res.statusText
          );
        } else {
          alert(`Done! See the uploaded file in the /static directory.`);
        }
      })
      .catch((err) => alert("Ooops: " + err));
  }
</script>

<svelte:head>
  <link
    href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/2.0.0-alpha.2/cropper.css"
    rel="stylesheet"
  />
  <script
    src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/2.0.0-alpha.2/cropper.min.js"
  ></script>
</svelte:head>

<div class="wrap py-4 max-w-xl mx-auto">
  <!--File upload button-->
  <div class="mt-8">
    <label
      class="flex justify-center w-full h-32 px-4 transition bg-white border-2 border-gray-300 border-dashed rounded-md appearance-none cursor-pointer hover:border-gray-400 focus:outline-none"
    >
      <span class="flex items-center space-x-2">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="w-6 h-6 text-gray-600"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
          stroke-width="2"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"
          />
        </svg>
        <span class="font-medium text-gray-600">
          Drop files to Attach, or
          <span class="text-blue-600 underline">browse</span>
        </span>
      </span>
      <input
        on:change={insertFile}
        type="file"
        accept="image/*"
        name="file_upload"
        class="hidden"
      />
    </label>

    <button
      on:click={uploadImages}
      type="button"
      class="mt-3 float-right inline-flex w-full justify-center rounded-md border border-transparent bg-red-600 px-4 py-2 text-base font-medium text-white shadow-sm hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2 sm:ml-3 sm:w-auto sm:text-sm"
      >Upload</button
    >
  </div>
  <!--File upload button-->

  <!--Cropped image list-->
  <div class="image-list mt-20">
    <ul />
  </div>
  <!--Cropped image list-->

  <!--Image modal-->
  <div
    id="image-edit-modal"
    class="hidden relative z-10"
    aria-labelledby="modal-title"
    role="dialog"
    aria-modal="true"
  >
    <div class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity" />
    <div class="fixed inset-0 z-10 overflow-y-auto">
      <div
        class="flex min-h-full items-end justify-center p-4 text-center sm:items-center sm:p-0"
      >
        <div
          class="relative transform overflow-hidden rounded-lg bg-white text-left shadow-xl transition-all sm:my-8 sm:w-full sm:max-w-lg"
        >
          <div class="bg-white px-4 pt-5 pb-4 sm:p-6 sm:pb-4">
            <div class="sm:flex sm:items-center justify-center">
              <div
                class="modal-content mx-auto flex flex-shrink-0 flex-col gap-4 items-center justify-center"
              >
                <div class="h-72 md:h-96 img-container">
                  <!-- svelte-ignore a11y-img-redundant-alt -->
                  <img alt="Picture" />
                </div>

                <div>
                  <button
                    on:click={() => cropper.rotate(-45)}
                    class="inline-flex w-full justify-center rounded-md border border-transparent bg-red-600 px-4 py-2 text-base font-medium text-white shadow-sm hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2 sm:ml-3 sm:w-auto sm:text-sm"
                  >
                    <Fa icon={faRotateLeft} />
                  </button>

                  <button
                    on:click={() => cropper.rotate(45)}
                    class="inline-flex w-full justify-center rounded-md border border-transparent bg-red-600 px-4 py-2 text-base font-medium text-white shadow-sm hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2 sm:ml-3 sm:w-auto sm:text-sm"
                  >
                    <Fa icon={faRotateRight} />
                  </button>
                </div>
              </div>
            </div>
          </div>
          <div class="bg-gray-50 px-4 py-3 sm:flex sm:flex-row-reverse sm:px-6">
            <button
              on:click={cropImage}
              type="button"
              class="inline-flex w-full justify-center rounded-md border border-transparent bg-red-600 px-4 py-2 text-base font-medium text-white shadow-sm hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-offset-2 sm:ml-3 sm:w-auto sm:text-sm"
              >Crop</button
            >
            <button
              on:click={closeModal}
              type="button"
              class="mt-3 inline-flex w-full justify-center rounded-md border border-gray-300 bg-white px-4 py-2 text-base font-medium text-gray-700 shadow-sm hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 sm:mt-0 sm:ml-3 sm:w-auto sm:text-sm"
              >Cancel</button
            >
          </div>
        </div>
      </div>
    </div>
  </div>
  <!--Image modal-->
</div>
