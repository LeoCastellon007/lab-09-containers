# Lab 09: Knowledge check

**Answer each question below it, in your own words, in one to three sentences.** Replace the `...` below **Your answer:** with your text.

## 1. Credentials in the image?

Assume you want to share your image with others and make it publicly available on `ghcr.io`, so that anyone can pull it and run the analysis. A teammate suggests adding `COPY .aws /home/app/.aws` to the `Dockerfile`, so that nobody has to type `-v ~/.aws:/home/app/.aws:ro` any more. (This is a thought experiment: keep your own package private.)

**Your answer:** Give two reasons why this is a bad idea. Think of who could then read your credentials, and of what happens when your Learner Lab session ends.

If the image is public, anyone can pull it and read `/home/app/.aws` inside it, so they would get my AWS keys and could use my account, for example to read my Atlas password from SSM. Also, Learner Lab credentials stop working when the session ends, as mine did earlier today, so keys baked into the image would be expired and the image would fail until I rebuilt and pushed it again with new ones.

## 2. Where did the Atlas password come from?

In Step 4, you ran the analysis once with `-v ~/.aws:/home/app/.aws:ro` only, without the three `-e` options. The container still wrote to your Atlas cluster.

*Hint:* Read `get_atlas_settings()` in `src/book_analysis/analyze.py`.

**Your answer:** Explain, step by step, where the container found the Atlas settings in that run.

Without the `-e` options, `MONGODB_ATLAS_URL` is not in the container's environment, so `get_atlas_settings()` asks AWS Systems Manager (SSM) for the parameter `mongodb_credentials` in `us-east-1` instead. It can talk to AWS because I mounted my `~/.aws` folder at `/home/app/.aws`, where boto3 finds my Learner Lab credentials. The parameter's JSON value holds the Atlas `url`, `user`, and `password`, which the container then used to connect.

## 3. Same tag, different image

Run `docker images book-analysis`, and note the image ID of `book-analysis:0.1.0`. Then change the `description` in the `LABEL` of your `Dockerfile`, build again with the same command, `docker build -t book-analysis:0.1.0 .`, and run `docker images book-analysis` again.

**Your answer:** Paste the image ID from before and after the rebuild. Why is it a problem for someone who uses your image that `0.1.0` now names a different image, and what should you have done instead?

Before the rebuild `book-analysis:0.1.0` had the image ID `cbf8cdc7ac51`, and after it `734ea0a274de`. A teammate who already used or cited `0.1.0` now gets a different image under the same name, so they cannot reproduce their earlier runs or tell which image they have. For a changed image I should raise `version` in `pyproject.toml` (for example to `0.1.1`) and build it under that new tag, leaving `0.1.0` unchanged.
