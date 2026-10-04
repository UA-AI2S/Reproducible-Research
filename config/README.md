
In a reproducible AI research repository, a configuration file like `models.yaml` centralizes model metadata. This ensures that any researcher running your pipeline connects to the **exact same model version, region, and inference endpoints** with identical hyperparameters.

## Why these specific fields matter for replication:
* **revision_hash:** Hugging Face repositories can be updated at any time by their creators. Pinning the full Git commit SHA ensures your pipeline always pulls the exact model weights you used.
* **seed & temperature:** 0.0: Most closed-source APIs will not guarantee identical tokens across calls unless you set the temperature to zero and explicitly pass a fixed structural seed.
* **region:** Cloud providers update inference infrastructure incrementally. For sensitive runtime comparisons, routing traffic to the exact same AWS/GCP region ensures identical hardware environments.
* **Dated model_id:** Pointing to a generic alias like gpt-4o will instantly break reproducibility whenever the vendor swaps the underlying model backend.
