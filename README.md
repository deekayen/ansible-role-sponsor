# deekayen.sponsor

[![CI](https://github.com/deekayen/ansible-role-sponsor/actions/workflows/ci.yml/badge.svg)](https://github.com/deekayen/ansible-role-sponsor/actions/workflows/ci.yml) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-deekayen.sponsor-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/deekayen/sponsor/) [![Project Status: Concept – Minimal or no implementation has been done yet, or the repository is only intended to be a limited example, demo, or proof-of-concept.](https://www.repostatus.org/badges/latest/concept.svg)](https://www.repostatus.org/#concept) ![BSD 3-Clause license](https://img.shields.io/badge/license-BSD%203--Clause-blue)

An Ansible role that prints a request to sponsor [deekayen on GitHub Sponsors](https://github.com/sponsors/deekayen) in the play output. It is meant to be included from other roles. It changes nothing on the target host.

The role runs two `ansible.builtin.debug` tasks. The first prints one line picked at random from the `nags` list in `vars/main.yml` and reports `changed`, which notifies a handler. The handler, listening on `sponsor deekayen`, prints a boxed message naming `sponsor_name` and `sponsor_url` when handlers run.

## Requirements

- ansible-core 2.15 or newer on the controller.
- Nothing on the target. The role uses only `assert` and `debug`, so it needs no facts or privilege escalation.

## Supported platforms

| Platform | Versions |
| --- | --- |
| GenericBSD | all |
| GenericLinux | all |
| GenericUNIX | all |

CI lints the role, syntax-checks `tests/test.yml`, and runs it against localhost on `ubuntu-latest`.

## Installation

Galaxy lists `deekayen.sponsor` but has no imported release yet, so install it from git with `requirements.yml`:

```yaml
---
roles:
  - name: deekayen.sponsor
    src: https://github.com/deekayen/ansible-role-sponsor.git
    scm: git
    version: main
```

```bash
ansible-galaxy role install -r requirements.yml
```

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `i_sponsored` | `false` | Boolean. Set to `true` to skip the message and the handler. The comment in `defaults/main.yml` asks that you sponsor at any tier before you set it. |
| `sponsor_name` | `deekayen` | Name shown in the task name and the handler's boxed message. |
| `sponsor_url` | `https://github.com/sponsors/deekayen` | URL shown in the handler's boxed message. `tasks/assert.yml` fails the play unless it starts with `https://`. |

`vars/main.yml` holds `nags`, the list of lines the first task picks from. It is internal to the role.

## Behavior

- The message task sets `changed_when: true`, so every run reports at least one change unless `i_sponsored` is `true` or the play runs with `--skip-tags sponsor`.
- The boxed handler message has a fixed width, so a `sponsor_name` or `sponsor_url` of a different length than the defaults breaks the border alignment.

## Dependencies

None.

## Example playbook

Include it at the end of another role's tasks:

```yaml
---
- name: Ask for sponsorship.
  ansible.builtin.include_role:
    name: deekayen.sponsor
```

Set `i_sponsored: true` in the calling play, inventory, or extra vars to turn it off.

## Tags

| Tag | Tasks |
| --- | --- |
| `sponsor` | The message task and the handler. `--skip-tags sponsor` hides both. |
| `always` | Input validation in `tasks/assert.yml`. |

## Known issues

- The `nags` list in `vars/main.yml` hardcodes `https://github.com/sponsors/deekayen` as one of its lines. Overriding `sponsor_url` changes the handler's message but not that line.

## Development

CI runs on every push to `main` and every pull request (see `.github/workflows/ci.yml`). It runs `ansible-lint --profile production`, syntax-checks `tests/test.yml`, and then runs that playbook against localhost. To run the same checks locally:

```bash
pip3 install ansible-lint
ansible-lint --profile production
mkdir -p .ansible/roles && ln -sfn "$PWD" .ansible/roles/deekayen.sponsor
ANSIBLE_ROLES_PATH=.ansible/roles ansible-playbook --syntax-check tests/test.yml -i tests/inventory
ANSIBLE_ROLES_PATH=.ansible/roles ansible-playbook tests/test.yml -i tests/inventory
```

The repository also has a `.pre-commit-config.yaml`; run `pre-commit run --all-files` before pushing.

### Repository layout

| Path | Purpose |
| --- | --- |
| `tasks/main.yml` | Input validation and the message task. |
| `tasks/assert.yml` | `sponsor_url` check, tagged `always`. |
| `handlers/main.yml` | Boxed message, listening on `sponsor deekayen`. |
| `defaults/main.yml` | Every user-facing variable. |
| `vars/main.yml` | The `nags` message list. |
| `tests/` | Playbook and inventory used by CI. |

## Releases

Pushing a git tag runs `.github/workflows/release.yml`, which imports the tagged commit into Ansible Galaxy as `deekayen.sponsor`. The import needs a `GALAXY_API_KEY` repository or organization secret. No tag has been pushed yet.

## License

BSD 3-Clause. See [LICENSE](LICENSE).

## Author

[David Norman](https://github.com/deekayen). Sponsorship links are in [.github/FUNDING.yml](.github/FUNDING.yml).
