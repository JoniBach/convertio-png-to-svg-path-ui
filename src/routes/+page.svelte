<script>
  import { PUBLIC_API_KEY } from "$env/static/public";
  import axios from "axios";
  let file = null;
  let conversionId = null;
  let svgUrl = "";
  let isLoading = false;
  let error = "";

  const apiKey = PUBLIC_API_KEY; // Replace this with your actual API key

  const handleFileChange = (event) => {
    file = event.target.files[0];
  };

  const uploadFile = async () => {
    if (!file) {
      error = "Please select a PNG file to upload.";
      return;
    }

    try {
      isLoading = true;
      error = "";

      // Step 1: Start the conversion
      const response = await axios.post("https://api.convertio.co/convert", {
        apikey: apiKey,
        input: "upload",
        file: "", // Placeholder; the actual file will be uploaded in the next step
        outputformat: "svg",
      });

      if (response.data.status === "ok") {
        conversionId = response.data.data.id;
        await uploadBinaryFile(conversionId, file.name);
      } else {
        error = response.data.error;
      }
    } catch (err) {
      error = "Failed to initiate the conversion.";
    } finally {
      isLoading = false;
    }
  };

  const uploadBinaryFile = async (conversionId, filename) => {
    try {
      isLoading = true;
      const url = `https://api.convertio.co/convert/${conversionId}/${filename}`;

      // Step 2: Upload the binary file
      await axios.put(url, file, {
        headers: {
          "Content-Type": "application/octet-stream",
        },
      });

      // After uploading, check the status
      checkConversionStatus();
    } catch (err) {
      error = "Failed to upload the binary file.";
      isLoading = false;
    }
  };

  const checkConversionStatus = async () => {
    if (!conversionId) {
      return;
    }

    isLoading = true;
    try {
      // Step 3: Check the conversion status
      const response = await axios.get(
        `https://api.convertio.co/convert/${conversionId}/status`
      );

      if (response.data.status === "ok") {
        const { step, output } = response.data.data;
        if (step === "finish" && output.url) {
          svgUrl = output.url;
        } else if (step !== "finish") {
          // If not finished, keep checking every 3 seconds
          setTimeout(checkConversionStatus, 3000);
        }
      } else {
        error = response.data.error;
      }
    } catch (err) {
      error = "Failed to check conversion status.";
    } finally {
      isLoading = false;
    }
  };
</script>

<main>
  <h1>PNG to SVG Converter</h1>

  <input type="file" accept=".png" on:change={handleFileChange} />
  <button on:click={uploadFile} disabled={isLoading}>
    {isLoading ? "Converting..." : "Upload and Convert"}
  </button>

  {#if error}
    <p style="color: red;">{error}</p>
  {/if}

  {#if svgUrl}
    <p>
      Conversion Successful! <a href={svgUrl} target="_blank">Download SVG</a>
    </p>
    <img
      src={svgUrl}
      alt="Converted SVG"
      style="max-width: 100%; height: auto;"
    />
  {/if}
</main>

<style>
  main {
    text-align: center;
    padding: 2rem;
  }

  input {
    margin-bottom: 1rem;
  }

  button {
    background-color: #007bff;
    color: white;
    border: none;
    padding: 0.5rem 1rem;
    cursor: pointer;
    font-size: 1rem;
    margin-bottom: 1rem;
  }

  button:disabled {
    background-color: #ccc;
  }
</style>
