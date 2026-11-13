# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

Personal sandbox for learning Kubernetes. Nothing here is production, so favour small, readable manifests over polish. When explaining a concept, tie it to a manifest or command in the repo where one exists. Keep learning notes in `notes/*.md` as short bullets, not long walkthroughs.

## State of the repo

Fresh repo with a single initial commit. It only contains `README.md` (just the `# k8s` title), `LICENSE`, and a `.gitignore`. There is no source code, build system, tests, or CI yet, so there are no build/lint/test commands to document.

## What the `.gitignore` implies

It is the stock Terraform template (ignores `.terraform/`, `*.tfstate*`, `*.tfvars*`, `override.tf*`, `.terraformrc`), so Terraform is expected to land here. `*.tfvars` is ignored, so variable values must not be committed — pass them in at plan time or document required inputs in `variables.tf`.

Update this file with real commands and architecture once the first manifests, modules, or scripts are added.
