# Introduction #

```
# Get a list of enabled/active modules.
[admin@server ~]# 

    source ~/local-settings.txt 

    drupalsite=sanctuary

        for modname in $(drush --pipe -l "${drupalsite?}" pm-list --no-core --status=enabled)
        do

            modinfo=$(
                drush -l "${drupalsite?}" pm-info "${modname?}" \
                    | sed -n '
                        /^[[:space:]]*Extension/{
                            s/^[[:space:]]*Extension[[:space:]]\{1,\}:[[:space:]]\{1,\}\([^[:space:]]\{1,\}\)[[:space:]]*/\1/
                            h
                            }
                        /^[[:space:]]*Project/{
                            s/^[[:space:]]*Project[[:space:]]\{1,\}:[[:space:]]\{1,\}\([^[:space:]]\{1,\}\)[[:space:]]*/\1/
                            H
                            }
                        /^[[:space:]]*Version/{
                            s/^[[:space:]]*Version[[:space:]]\{1,\}:[[:space:]]\{1,\}\([^[:space:]]\{1,\}\)[[:space:]]*/\1/
                            H
                            x
                            s/\n/ /g
                            p
                            }
                        '
                    )

            modextn=$(echo ${modinfo?} | cut -d ' ' -f 1)
            modproj=$(echo ${modinfo?} | cut -d ' ' -f 2)
            modvers=$(echo ${modinfo?} | cut -d ' ' -f 3)

            echo "  Extn [${modextn?}]"
            echo "  Proj [${modproj?}]"
            echo "  Vers [${modvers?}]"

            if [ "${modproj?}" != 'drupal' ]
            then

                if [ "${modproj?}" != 'Unknown' ]
                then

                    if [ "${modproj?}" ==  "${modextn?}" ]
                    then
                        echo "drush -l ${modproj}-${modvers}"
                    fi
                fi
            fi

        done

```