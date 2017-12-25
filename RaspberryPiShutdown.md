# Raspberry Pi Shutdown/Reset/Start Button

Posted in [Tutorials](/categories/#)  and tagged [Raspberry Pi](/tags/#Raspberry Pi "Raspberry Pi") on Jan 25, 2015

</header>

<article class="post-content">

Shutting down a Raspberry Pi by cutting the power while it is still running is not recommended and it can lead to data corruption. The Raspberry Pi does not have a built-in shutdown/reset button, but thankfully it is fairly simple to wire one up.

### Wiring up the pushbutton

<small>_Note: info below applies to B+ and probably rev 2 A/B boards._</small>

Take a pushbutton and simply wire it up as seen in the diagram below, one leg going via the red wire to **pin 5 (GPIO3)** and the other side going to ground at **pin 6 (GND)**. A **pull-up resistor** is also needed on the wire connected to **pin 5** (see e.g. [here](https://learn.sparkfun.com/tutorials/pull-up-resistors) for reason) but the same can be achieved by just enabling the built-in pull-up resistor in code (see below).

![Wiring the pushbutton](/images/pi-shutdown-button.png)

Pins other than number 5 could also be used, however a side benefit of using it is that when the Pi is powered off, shorting pin 5 with ground causes the Raspberry Pi to power on. So the **same button can be used for shutdown/reboot and also for power-on**.

### Code

The code to control the button is fairly simple. First set up the GPIO pin:

<div class="language-python highlighter-rouge">

    GPIO.setmode(GPIO.BOARD)
    GPIO.setup(5, GPIO.IN, pull_up_down=GPIO.PUD_UP)

</div>

Then, subscribe to button up/down events:

<div class="language-python highlighter-rouge">

    GPIO.add_event_detect(5, GPIO.BOTH, callback=buttonStateChanged)

</div>

Finally, in the event handler act on the button press:

<div class="language-python highlighter-rouge">

    def buttonStateChanged(pin):
        global buttonPressedTime

        if not (GPIO.input(pin)):
            # button is down
            buttonPressedTime = datetime.now()
        else:
            # button is up
            if buttonPressedTime is not None:
                if (datetime.now() - buttonPressedTime).total_seconds() >= shutdownMinSeconds:
                    # button pressed for more than specified time, shutdown
                    call(['shutdown', '-h', 'now'], shell=False)
                else:
                    # button pressed for a shorter time, reboot
                    call(['shutdown', '-r', 'now'], shell=False)

</div>

Using this code, if the button is pressed for more than a few seconds then the Raspberry Pi **shuts down**, otherwise it **reboots**.

The full script is available at [https://github.com/gilyes/pi-shutdown](https://github.com/gilyes/pi-shutdown)

### Run the script

<div class="highlighter-rouge">

    sudo python pishutdown.py

</div>

### Autostart the script

If you’re using `systemd` then create a file called `pishutdown.service` in `/etc/systemd/system/` (replace `_path\_to\_pishutdown_` with appropriate path):

<pre>[Service]
ExecStart=/usr/bin/python /_path_to_pishutdown_/pishutdown.py
WorkingDirectory=/_path_to_pishutdown_/
Restart=always
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=pishutdown
User=root
Group=root

[Install]
WantedBy=multi-user.target
</pre>

Enable service:

<div class="highlighter-rouge">

    sudo systemctl enable pishutdown.service

</div>

Run service (will be automatically started on next reboot):

<div class="highlighter-rouge">

    sudo systemctl start pishutdown.service

</div>

</article>

* * *

</div>
