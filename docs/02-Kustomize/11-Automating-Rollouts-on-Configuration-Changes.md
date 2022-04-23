# Automating Rollouts on Configuration Changes

Now the the ConfigMap information is embedded in the Deployment information every time you modify the `options.env` file a new RollOut will be triggered:

Regenerate the options file:

```sh
cat > deploy/vote/staging/options.env << EOF
OPTION_A=Thalia
OPTION_B=Paulina
EOF
```
