#PRD as of 20251207

What are you really trying to do



Core User Journey

1. User installs the app via appstore or an apk
2. Screen 1:The app opens with message "totally offline app & optional updates"
    - Proceed button
3. Screen 2: User sees 7 blocks which are buttons with names of 7 twitter creators, last of which is unclassified, default will be Naval, Shreyas, Nikita Bier
    - Screen 3: User clicks on a block and goes to Screen 4
    - Screen 4: User sees tweet embeds of the given creator whose block had been clicked
        - User clicks on tweet embed is taken to
            - either default web browser
            - or twitter app if installed
            - at either places if the twitter app requires user to sign in, they would need to sign in to view the full tweet
4. Screen 2 has a bottom button of edit configuration
    - Screen 5: User can edit the
        - 7 creator names
        - tweets under each creator via URL
5. If user updates the app
    - current config is put into back up
    - new config is downloaded - but it will always have only 3 creators by default

