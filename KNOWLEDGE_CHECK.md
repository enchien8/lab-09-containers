# Lab 09: Knowledge check

**Answer each question below it, in your own words, in one to three sentences.** Replace the `...` below **Your answer:** with your text.

## 1. Credentials in the image?

Assume you want to share your image with others and make it publicly available on `ghcr.io`, so that anyone can pull it and run the analysis. A teammate suggests adding `COPY .aws /home/app/.aws` to the `Dockerfile`, so that nobody has to type `-v ~/.aws:/home/app/.aws:ro` any more. (This is a thought experiment: keep your own package private.)

**Your answer:** Give two reasons why this is a bad idea. Think of who could then read your credentials, and of what happens when your Learner Lab session ends.

Anyone who can pull a public image could extract the copied AWS credentials, exposing my account access. Learner Lab credentials expire, so credentials baked into the image would eventually stop working and could not be refreshed without rebuilding the image.

## 2. Where did the Atlas password come from?

In Step 4, you ran the analysis once with `-v ~/.aws:/home/app/.aws:ro` only, without the three `-e` options. The container still wrote to your Atlas cluster.

*Hint:* Read `get_atlas_settings()` in `src/book_analysis/analyze.py`.

**Your answer:** Explain, step by step, where the container found the Atlas settings in that run.

With no `-e` options, the container had no `MONGODB_ATLAS_URL`, so `get_atlas_settings()` used its SSM path. The read-only `~/.aws` mount supplied AWS credentials to `boto3`, which retrieved and decrypted the `mongodb_credentials` parameter; the code parsed its JSON to get the Atlas URL, user, and password.

## 3. Same tag, different image

Run `docker images book-analysis`, and note the image ID of `book-analysis:0.1.0`. Then change the `description` in the `LABEL` of your `Dockerfile`, build again with the same command, `docker build -t book-analysis:0.1.0 .`, and run `docker images book-analysis` again.

**Your answer:** Paste the image ID from before and after the rebuild. Why is it a problem for someone who uses your image that `0.1.0` now names a different image, and what should you have done instead?

Before the rebuild, the image ID was `8048c5f9a94b`; after changing the description and rebuilding under `0.1.0`, it was `3f7132b0762c`. Reusing a published version tag for different image contents makes it unclear which image a user will receive; I should have left `0.1.0` unchanged and used a new version tag, such as `0.1.1`.
