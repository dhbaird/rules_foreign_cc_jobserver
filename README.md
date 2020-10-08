# rules_foreign_cc_jobserver

Setup for systemd-based system:

```
sudo cp --preserve=mode start_jobserver /usr/local/bin
cp jobserver.service ~/.config/systemd/user/jobserver.service
systemctl --user daemon-reload
systemctl --user enable --now jobserver
systemctl --user status jobserver.service
```
