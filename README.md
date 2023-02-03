# Shelly-1PM---graph-and-summary-of-consumption
This pythoncode, will generate a live graph that shows the average and total powerconsumption of any device connected to a Shelly 1PM, by pulling it from a Shelly 1PM.
Change the Device IP to reflect your Shelly 1PM's IP
device_ip = "IP of shelly 1PM" 
examply:
device_ip = "192.168.0.99" 

---------------------

    import requests
    import time
    import matplotlib.pyplot as plt
    from matplotlib.animation import FuncAnimation

    device_ip = "192.168.0.99" # replace with the actual IP of your Shelly1/1PM device
    maxpower = 0
    start_time = time.time()
    firstrun = True;
    aenergystart = 0;
    aenergycurrent = 0;

    power_consumption_values = []

    fig, ax = plt.subplots()
    ax.set_xlabel("Sample")
    ax.set_ylabel("Power Consumption (W)")
    ax.set_ylim(0, None)
    line, = ax.plot([], [], "-o")

    def update(frame):
        global firstrun, aenergycurrent, aenergystart, maxpower, power_consumption_values, start_time
        response = requests.get(f"http://{device_ip}/rpc/shelly.getstatus")

        if response.status_code == 200:
            data = response.json()
            current_power_consumption = data["switch:0"]["apower"]
            if firstrun:
                aenergystart = data["switch:0"]["aenergy"]["total"]
                firstrun = False
            else:
                aenergycurrent = data["switch:0"]["aenergy"]["total"]

            power_consumption_values.append(current_power_consumption)
        else:
            print("Failed to retrieve power consumption information")

        line.set_data(list(range(len(power_consumption_values))), power_consumption_values)
        ax.relim()
        if current_power_consumption > maxpower:
            maxpower = current_power_consumption + 1
        ax.set_ylim(0, maxpower)
        ax.autoscale_view()

        elapsed_time = time.time() - start_time
        average_power_consumption = sum(power_consumption_values) / len(power_consumption_values)
        total_power_consumption = average_power_consumption * elapsed_time / 3600 / 1000
        consumedpower = (float(aenergycurrent) - float(aenergystart)) / 1000; 
        diff = float(aenergycurrent) - float(aenergystart);

        plt.suptitle(f"Time Elapsed: {int(elapsed_time)}s \nTotal Power Consumption (based on datapoints): {total_power_consumption:.3f} kWh\nAverage Power Consumption: {average_power_consumption:.3f} W\nTotal Power Consumption(Real): {consumedpower:.3f} kWh")

        print("DEBUGGING \n")
        print(total_power_consumption);
        print(consumedpower)


    ani = FuncAnimation(fig, update, interval=1 * 1000)
    plt.show()
