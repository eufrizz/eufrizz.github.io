---
layout: post
title:  "[DRAFT] Why is underwater robotics so difficult?"
date:   2024-06-03 14:20:24 +0
categories: robotics marine underwater
comments: true
excerpt: Marine robotics is an area with huge potential for impact, but most people have very little idea about it. Here I explain some of the unique and interesting challenges that come with creating a robot for an underwater environment.
assets: /assets/images/underwater-robots/
preview-image: "/assets/images/underwater-robots/hullbot_under_hull.png"
---

*I was a software engineer at Hullbot, a marine robotics startup, for 3 years. This is an overview of some of the unique challenges present in building a robot in the underwater space!*

<div style="display:flex" height=200>
  <div style="flex: 50%; padding: 4px; overflow:hidden" >
    <a href="https://www.hullbot.com" target="_blank">
        <img src="{{page.assets}}/hullbot.jpg" alt="Hullbot underwater robot" title="Hullbot" style="width:100%" >
    </a>
    <a href="https://bluerobotics.com" target="_blank">
        <img src="https://bluerobotics.com/wp-content/uploads/2016/06/BlueROV2.jpg" alt="Blue robotics ROV" title="BlueRobotics BlueROV2" style="width:100%" >
    </a>
  </div>
  <div style="flex: 50%; padding: 4px; overflow:hidden">
    <a href="https://www.anduril.com" target="_blank">
        <img src="https://images.marinetechnologynews.com/images/maritime/w800h500/photo-courtesy-anduril-133545.jpg" alt="Anduril Ghost Shark XLUUV" title="Anduril Ghost Shark"  style="width:100%">
    </a>
    <a href="https://www.ifremer.fr/fr/flotte-oceanographique-francaise/decouvrez-les-navires-de-la-flotte-oceanographique-francaise/victor-6000" target="_blank">
        <img src="https://data.ibtimes.sg/en/full/68424/victor-6000.jpg?w=736" alt="Victor 6000" title="Ifremer Victor 6000" style="width:100%">
    </a>
  </div>
</div>
<div class="caption"> Marine robots! Click to find out more.</div>



Robots and artificial intelligence are popping up in more places than ever before - in drones, as self-driving cars, as vacuum cleaners, in factory production lines. But one medium has gained significantly less attention: marine robots. On the surface (no pun intended), making an underwater robot would seem as challenging as any other area, but it does in fact come with its own unique set of technical challenges that make it surprisingly difficult. These fall roughly into four categories: mechanical, autonomy, communication and business. Any company in the area is acutely aware of these issues, and how they choose to tackle them defines the product. And by tackling them, they begin to unlock the vast potential of exploration and utilisation of 71% of the earth’s surface.

<div class="six-image-grid">
    <a href="https://www.skydio.com" target="_blank">
        <img src="https://cdn.sanity.io/images/mgxz50fq/production-v2/fe2471c594d6333e77433bc6c5dc6fc28aab4172-1920x1095.jpg?w=1920&h=1095&auto=format" alt="Skydio drone" title="Skydio UAV">
    </a>
    <a href="https://covariant.ai" target="_blank">
        <img src="https://www.materialhandling247.com/images/2021_images/MH247_Covarient_inline3_800p.png" alt="Covariant arm" title="Covariant factory arm">
    </a>
    <a href="https://waymo.com" target="_blank">
        <img src="https://images.ctfassets.net/e6t5diu0txbw/4rGbzVdkyNAnPEoqFyVpdx/b36b99fa83e9f6b662b3be4ae8516826/Waymo_SF.jpg" alt="Waymo self-driving car" title="Waymo self-driving car">
    </a>
    <a href="https://www.figure.ai" target="_blank">
        <img src="https://techstory.in/wp-content/uploads/2024/03/Jeff-Bezos-Nvidia-join-OpenAI-in-funding-humanoid-robot-startup-Figure-AI.webp" alt="Figure humanoid robot" title="Figure AI humanoid">
    </a>
    <a href="https://maticrobots.com" target="_blank">
        <img src="{{ page.assets }}/Matic.jpg" alt="Matic robot vacuum" title="Matic robot vacuum">
    </a>
    <a href="https://bostondynamics.com" target="_blank">
        <img src="https://bostondynamics.com/wp-content/uploads/2023/06/spot-charging-dock-industrial-min.jpg" alt="Boston Dynamics spot robot dog" title="Boston Dynamics Spot">
    </a>
</div>
<div class="caption"> Some robots that have gained widespread attention or use. Click to find out more.</div>

It may help to first give an overview of the broad range of environments and applications in the space. We can start with surface vehicles, which are basically autonomous boats, powered or sailing, which often survey large bodies of water, or transport passengers and cargo. Underwater robots tend to be either shallow - for operating in coastal areas, on reefs or on boats - or deep sea, for inspecting, maintaining or creating deep sea cables or drilling. The common environmental challenges are the harsh conditions, where water, wind, waves, currents and pressure make it difficult to survive and are highly variable. 

### Mechanical

The mechanical issues in underwater robotics stem fundamentally from the fact that water and electronics don’t mix. Most robots have a “brain”, or central computer, that connects to external sensors (e.g. cameras) and actuators (e.g. motors), enabling interaction with its environment. As you will know from spilling liquids on your laptop or dropping your phone into the ocean, water, and especially salt water, destroy circuit boards.

There are two common ways to waterproof electronics - by coating them in a substance like epoxy or silicone - “potting” or “conformal coating” - or placing them in a watertight enclosure. Potting can be super simple - just smother the electronics in epoxy - but there’s no going back and taking it off for maintenance or repair, so it’s not a great choice in the development stage, or for expensive, complex parts. On the other hand, watertight enclosures give you the flexibility of accessing the parts inside, but come at a higher cost as the enclosures require development and production. A good compromise is to buy an off the shelf enclosure, like the popular ones from [BlueRobotics](https://bluerobotics.com/product-category/watertight-enclosures/locking-series/).

<div style="display: block; text-align:center;">
<img src="http://aptelectronics.co.uk/images/Potting-Compound.jpg" style="width:28%; padding: 4px">
<img src="https://www.conro.com/wp-content/uploads/2021/08/conformal-coating-types.jpg" style="width:40%; padding: 4px">
</div>
<div class="caption"> Potting on the left, conformal coating on the right. Conformal coating tends to be less heavy duty and unsuited to prolonged immersion.</div>

Now, that may seem like problem solved, but water is in fact a cunning enemy that will do anything to permeate what you once presumed were your impenetrable defences. Your potting will be compromised by the wrong match of potting material and surface to bind to, bad surface preparation, air bubbles, curing, sneaky corners you forget to fill, movement, wicking through wires, and on and on. You then may only find out about it a month later, when after continued immersion your electronics start to mysteriously misbehave due the tiniest, slowest leak that has finally managed to wrangle its way in. If you’re lucky, you then realise you have a design flaw, and you can chuck out the batch of parts you potted that time because the rest of them will eventually fail in the same way, and try fix the issue for the next iteration. Most of the time, you don’t even realise the issue is due to water ingress and you go off blaming the electronics, or you suspect water but you’re stuck not being able to diagnose it because all you’ve got is a lump of goo that you can’t see into.

An enclosure, usually sealed with a gasket or O-ring, tends to be a bit more forgiving in diagnosing and fixing leaks because you can pull it apart and see where the water is getting in. You can immersion test enclosures thoroughly before putting your expensive electronics in, simply checking for water ingress after a period of sustained immersion, or using a moisture detector. Pressure testing can also be done by pressurising or vacuuming the enclosure, and tracking the change in pressure over time with a sensor - this is probably the most robust technique for checking for leaks.

If you discover that your enclosure is leaky, you then need to pinpoint where it is occurring. Enclosures usually have a number of ports that provide connections to external electronics - think motors, sensors that require specific positioning, or a data cable to the operator on the surface. These external devices and their cables all need to be watertight too, or you’ll end up with water entering the device, then wicking its way down the cables and into the main enclosure.  Something as simple as a cut on a cable can spell a major leak. A complex robot with more external devices means more points of failure, and more difficulty in diagnosing a leak.

<div style="display: block; text-align:center;">
<img src="https://www.roboticocean.com/wp-content/uploads/2019/05/DSC1702_-scaled.jpg" style="width: 60%">
</div>
<div class="caption"> A typical enclosure for electronics. 5 ports with watertight connectors for electronics are visible. Courtesy of <a href="https://www.roboticocean.com">RoboticOCEAN</a></div>

Leak diagnosis can be a mundane process of sealing one port at a time and doing a pressure test to find the offending device, which may take a while with, say, 16 ports. Acoustic leak detection is another handy tool that amplifies the ultrasonic vibrations that the particles escaping a leak create, allowing you to pinpoint invisible flaws in your enclosure. As an example of one insidious leak, a manufacturer once sent a batch of custom enclosures. On one of them, they had drilled just a fraction of a millimetre too far into one screw hole, causing a minor weakness in the aluminium that was enough for it to fail a pressure test and cause a leak. It took a whole day of triple checking every port in the enclosure, and the main gasket, before finally pinpointing this invisible and completely unexpected flaw with an acoustic leak detector.

Another important design factor for an enclosure is water pressure. For every 10m in depth, roughly an extra atmosphere of pressure is added, and this builds up to insane pressures and forces in deep sea environments. Gaskets and enclosures have to be designed to withstand the pressure, and this adds cost as they become heavier duty. This is also the reason why most enclosures are cylindrical, as circular shapes distribute the forces of pressure most evenly.

Aside from the R&D needed to attain a watertight robot, the requirement for watertightness adds inflexibility to the development and design of the robot. On an aerial or terrestrial robot, you can slap on a new sensor or change the position of a camera without much effort, quickly evaluate its effectiveness, and iterate towards an optimal design. However, you can’t simply plug and play with new sensors underwater as they have to integrate into a watertight system. Most sensors or actuators are not watertight off the shelf, and you need a solution for waterproof cabling/connections. All of this increases the time and cost of development. The compromise that most projects settle on is to put as much of the electronics into a central enclosure as possible, and have standardised waterproof connectors to any devices that need to be external to the enclosure, finding a balance between flexibility and cost.

<div style="display: block; text-align:center;">
<img src="https://www.rotor-magazin.com/wp-content/uploads/2018/08/basis_motoren.jpg" style="width: 35%">
<img src="http://bearingfailureanalysis.co.uk/assets/faq/16-rust-and-corrosion/thumb/photo-44.jpg" style="width: 25%">
<img src="http://steelfabservices.com.au/wp-content/uploads/2017/04/Galvanic-corrosion.jpg" style="width: 37%">

</div>
<div class="caption"> Left: brushless motor internals - unfortunately I don't have a photo of a corroded one. Middle: corroded bearing. Right: galvanic ("dissimilar metal") corrosion</div>

Reliability is also impacted by being in a marine environment, due to the highly corrosive effects of saltwater on metal. Anodised aluminium and polymers/plastics are common material choices that stand up to corrosion, but sometimes they are not suitable. Motors are a good example - they require magnets to operate, which are iron-based and highly susceptible to corrosion. The bearings are also typically made of hardened steel, and these seize up with prolonged exposure to saltwater. Galvanic corrosion is also a big issue, where two different metals in the presence of saltwater will create an electrical current, causing one to dissolve at an accelerated rate. Stray voltages from the electronics connected to any metal enclosure will also cause electrolysis of the metal in saltwater. Thus, careful and often more expensive design choices need to be made to prevent corrosion, often in combination with maintenance like rinsing in freshwater and applying anti-corrosion lubricants.

### Autonomy (localisation and navigation)

<!-- ![Video]({{page.assets}}/murky_h5.mp4) -->
<div style="text-align: center;">
<video width="75%" controls loop playsinline>
  <source src="{{page.assets}}/murky_h5_donald.mp4" type="video/mp4">
  <!-- Not sure if webm is necessary -->
  <source src="{{page.assets}}/murky_h5_donald.webm" type="video/webm">
</video>
</div>
<div class="caption"> As <a href="https://www.youtube.com/watch?v=-6CEryWus_A">Donald Byrd</a> puts it, where are we going? Underwater vision sucks!</div>

Once the physical frontier is conquered with a robot that can withstand the marine environment, the issue of navigating it rears its head. In our daily lives we take for granted the crystal clarity of air, and our eyes have evolved to be our primary sensor for navigating our world. But anyone who’s ever dived or snorkelled quickly becomes aware of how impeded vision is in underwater environments. Most of the time, turbidity (particulate matter in the water) causes a murkiness that restricts line of sight to anywhere from 10m (on a good day) to 10cm (on a bad day). Red light is also attenuated more strongly, causing everything to turn blue-green. And all light is absorbed, causing things to quickly get dark the deeper you go (but this is fairly straightforward to overcome by adding lights to your robot). The generally poor, and highly variable visual conditions have a number of consequences for localisation - that is, knowing your location and the surrounding environment.

Firstly, being surrounded by turbidity simply means that any landmarks further than a couple of meters away become invisible. It’s very easy to get lost in the haze and become completely disoriented - often you can’t see the sea floor or the nearest submerged structure, so you have no idea which way to go to get back to where you were. The particulate matter in the water creates a lot of noise, which is often worsened by your own lights strongly illuminating the randomly moving particles in front of the camera. Not only does light attenuation reduce the range of your lights, but if you try to overcome this with stronger lights, you just end up illuminating the particles, creating more noise, and having less chance of picking up anything illuminated by ambient light. 


<div style="display: grid; grid-template-columns: repeat(2, 1fr); text-align:center; gap: 5px 5px;">
<img src="{{ page.assets }}/murky_1_crop.jpg" style="">
<img src="{{ page.assets }}/medium_viz_crop.jpg" style="">
<img src="{{ page.assets }}/bubbles_crop.jpg" style="">
<img src="{{ page.assets }}/contrast_2_crop.jpg" style="">

</div>
<div class="caption"> A range of underwater visual scenarios. Top left: poor visibility with a couple of bubbles. Top right: medium visibility. Bottom left: Many bubbles. Bottom right: high contrast and refractions on the left</div>


There are other weird visual scenarios in water. A few bubbles can add some misleading noise as they move around the frame, and these outliers can be fairly easily rejected, but a lot of bubbles will obscure your view entirely - this is a common occurrence at the surface. Refraction with the water surface also adds weird effects that can be very difficult to interpret. Lighting can also be extreme when you have direct sunlight in the lens but you’re trying to inspect the dark underside of an object.

This all means that all the useful computer vision and localisation techniques like [optical flow](https://learnopencv.com/optical-flow-in-opencv/), [feature detection](https://medium.com/@deepanshut041/introduction-to-feature-detection-and-matching-65e27179885d), [object detection](https://en.wikipedia.org/wiki/Object_detection), [semantic segmentation](https://encord.com/blog/guide-to-semantic-segmentation/), and [SLAM](https://www.mathworks.com/discovery/slam.html), which are highly developed and highly effective for aerial and terrestrial robot navigation, end up breaking down underwater. Feature-based [visual odometry](https://en.wikipedia.org/wiki/Visual_odometry) techniques, which assign small patches of the image a digital “fingerprint” so they can be tracked between video frames, are generally very unreliable due to the noise.  “Direct” vision techniques, which track the movement of image texture, are more robust to the noise.

This is not to say that cameras are useless underwater - they are in fact very useful, but vision algorithms underwater need much more sophistication to extract signal from the noise. It also necessitates greater reliance on other sensors, and intelligent ways of fusing their data that can tell when the vision quality is poor. But the more you dig into underwater sensing, the clearer it becomes that all of the incredible progress that has led powerful technologies like cameras, GPS, LiDAR and WiFi to become so cheap and enable the spread of above-surface robots, has not transferred to marine robots anywhere near as effectively.

LiDAR and GPS, the most commonly used above-surface sensors for sensing a robot’s environment, are severely affected underwater. The absorption of radio waves in water (the same reason a microwave oven works) means that GPS doesn’t penetrate water more than a few centimetres. LiDAR suffers from the same issues as a camera, namely light absorption (which can be helped by choosing a laser wavelength in the blue spectrum), and turbidity. Additionally, LiDAR is expensive, and any of the [cheaper units](https://www.robotshop.com/products/rplidar-a1m8-360-degree-laser-scanner-development-kit) for above-surface applications are unsuitable as they use infrared light (which is highly attenuated in water), and their spinning parts are not waterproof. No off the shelf or remotely cheap solutions exist for underwater purposes. The next best thing is a [time of flight (TOF) sensor](https://en.wikipedia.org/wiki/Time-of-flight_camera), which uses a single beam of light to measure distance, traditionally in only one axis, but there are now ones which can create a 2D depth map. However, off the shelf solutions have a short range in the low to mid tens of cm underwater.

<div style="display: block; text-align:center;">
<img src="https://www.rovplanet.com/tportal_upload/md_weblap_tartalom/pic2_1986.jpg" style="width: 32%">
<img src="https://bluerobotics.com/wp-content/uploads/2023/07/ACOUSTIC-POSITIONING-SYSTEM-USBL.jpg" style="width: 31%">
<img src="{{ page.assets }}/DVL1000-4000_4.jpg" style="width: 33%">

</div>
<div class="caption"> A vareity of acoustic sensors! Left: <a href="https://www.rovplanet.com/words-smallest-imaging-sonar-to-be-launched-at-oceanology-international-2020-04-03-2020">scanning sonar</a>. Middle: <a href="https://bluerobotics.com/learn/a-smooth-operators-guide-to-underwater-sonars-and-acoustic-devices/">USBL positioning system</a>. Right: <a href="https://www.nortekgroup.com/products/dvl1000-6000m">DVL</a> with acoustic beams visualised</div>

Acoustic sensors are a technology that do overcome many of these limitations of the electromagnetic spectrum, and are very useful in marine environments. The basic technique is to send a pulse of sound/vibration and measure the reflection, which can tell you the distance to the objects and even their shape. As a brief overview:

- Echosounders use reflections off the sea floor to tell its depth and contours. A [single beam echosounder](https://ceruleansonar.com/products/sounder-s500) can cost a few hundred dollars and weigh ~100g, but can only tell you distance to the bottom. A multi beam echo sounder is the gold standard for sea floor mapping, but they cost tens of thousands of dollars and are fitted to ships.
- Sonars are a similar technology, but are used to detect objects in all directions, not just the sea floor. Scanning sonars rotate the transducer to map out a section of interest or a whole 360°, but are slow to update and cost a few thousand dollars. Side scan sonars are similar but tend to focus on the sea floor. Mutibeam sonars are fast to update but are much more complex and costly (basically un ultrasound machine).
- Acoustic positioning systems use transducers in fixed locations to triangulate the position of a vessel, kind of like an underwater GPS. This of course requires infrastructure, whether mounted to a ship, nearby structure or sea floor.
- Doppler velocity loggers (DVLs) provide an accurate way to measure velocity relative to the sea floor, measuring the shift in frequency of reflections due to the doppler effect. As they can’t give absolute position, they do drift over time, but they tend to be very accurate in the short to medium term. They tend to be a couple of hundred grams and cost a few thousand dollars.

(For more detail, check out this great [article](https://bluerobotics.com/learn/a-smooth-operators-guide-to-underwater-sonars-and-acoustic-devices/) from Blue Robotics.)

DVLs, acoustic positioning systems and sonars are the most common sensors fitted to robots, and their use varies widely depending on the application. Sonars are great for detecting nearby objects, and DVLs and acoustic positioning are great for positioning. However, they are generally much more expensive and heavier/larger than sensors you would find on above-surface robots, which is why you never see them used outside of marine environments (i.e. in air you can get away with more effective, cheaper sensors, so you’d never bother with acoustic). Cost is the prohibitive factor in making robots with these sensors scalable.

Interesting quirks of underwater robotics also come from the dynamics of moving through the dense medium of water. Hydrodynamic drag is significant, and means that marine robots are limited in speed and range. However, it does add dampening and stability, making dynamics and control a much more forgiving problem, and preventing any high speed crashes. Buoyancy is also a very useful effect, allowing a well-designed, neutrally buoyant robot to simply hover at any depth with almost no energy expenditure. It also makes development very safe, as robots won’t drop out of the sky and injure humans - in the worst case, they will glide to the bottom of the ocean, or rise to the top. Ocean currents are an issue, however, and require a lot of energy to fight due to drag. They also mean that a robot will quickly drift if it has no reference from a nearby structure or the sea floor. It’s not uncommon to be surveying an underwater structure, lose visual tracking momentarily due to bad visual conditions or execution of a manoeuvre, and then have drifted away into the haze with no idea of which way you went.

<div style="display: block; text-align:center;">
<img src="https://cdn11.bigcommerce.com/s-2fbyfnm8ev/images/stencil/1280x1280/products/901/3151/apiady93f__08656.1548549981__68959.1586537920.jpg?c=2" style="width: 20%">
<img src="https://www.mouser.com/images/mcube/images/MTI-xx_new_xsens_SPL.jpg" style="width: 20%">
<img src="{{ page.assets }}/AeroBT-HG9900-front-pg_RHreflect_2880x1440-1.jpeg" style="width: 20%">

</div>
<div class="caption"> The range of IMUs. Left: <a href="https://www.mouser.com/ProductDetail/Bosch-Sensortec/BNO055?qs=QhAb4EtQfbV8Z2YmISucWw%3D%3D">Consumer grade ($12)</a>. Middle: <a href="https://www.mouser.com/ProductDetail/Movella-Xsens/MTI-30-4A8G4?qs=bZr6mbWTK5mz%2FK6UaMrm%2FA%3D%3D">Industrial grade ($1800)</a>. Right: <a href="https://aerospace.honeywell.com/us/en/products-and-services/product/hardware-and-systems/sensors/hg9900-inertial-measurement-unit">Navigation grade (>$100,000)</a>
</div>

Inertial/magnetic sensors are the last type of sensor we’ll cover, and their use is widespread. Accelerometers, gyroscopes and magnetometers allow the robot to know which way is up, how they are rotating, and how they are oriented relative to the north pole. They are an invaluable backup when other sensor modalities fail due to the challenging conditions, and allow for “[dead reckoning](https://en.wikipedia.org/wiki/Dead_reckoning#Autonomous_navigation_in_robotics)” to maintain stability and approximate positioning for a short period of time. Inertial measurement units (IMUs) can range from very small, inaccurate and cheap (on the order of dollars) consumer products, to large, accurate and expensive (on the order of hundreds of thousands of dollars), restricted to military and space applications like missiles - or anywhere in between. At a consumer price point, their usefulness for navigation alone is limited to a few seconds, but they do provide an accurate and high-frequency measurement of the robot’s orientation (roll/pitch/yaw) which is crucial for control.

### Communications

<div style="display: block; text-align:center;">
<img src="https://www.teledynemarine.com/en-us/products/PublishingImages/UCM_OEM_UCT.jpg" style="width: 20%">
<img src="https://reachrobotics.com/media/B3-Cutter-Defender.jpg" style="width: 40%">

</div>
<div class="caption"> Communications solutions. Left: <a href="https://www.teledynemarine.com/brands/benthos/acoustic-modems">Acoustic modem</a>. Right: <a href="https://reachrobotics.com/products/">Tethered robot</a>.
</div>

Communications is also a unique frontier for the underwater space, to due the absorption of radio waves. WiFi drops out in the first few centimetres. The solution for wireless communication is an acoustic modem, which uses ultrasound to broadcast a signal. However, these are limited to very low bandwidth, on the order of ~10 kbps - incapable of a video stream. They also cost [thousands](https://waterlinked.com/shop/modem-m16-186#attr=159), and have higher latency due to the slow speed of sound (1481m/s in water). Compare this to the [DJI FPV system](https://www.dji.com/dji-fpv/specs), which can do up to 120Mbps in air - a whopping 1000x difference. Thus, underwater robots must either do their work autonomously without the need for high bandwidth communication, or they must be tethered, i.e. they are connected to the operator by a wire for high speed data. A tether requires careful management to prevent it getting wrapped around objects, or sliced by nearby boats, but it also provides a very convenient way to retrieve your robot if it fails (which, in robotics, is common).

### Business

<div style="display: block; text-align:center;">
<img src="https://www.superyachtservicesguide.com/uploads/assets/listings/Sydney%20Superyacht%20Marina/2-Sydney%20Superyacht%20Marina-Callisto%20aerial%20photo%20with%20crane%20and%20fuel.jpeg" style="width: 45%">
<img src="https://schomberginv.com/img/about/about-img2.jpg" style="width: 36%">

</div>
<div class="caption"> Business costs. Left: Waterfront property. Right: A fouled (dirty) hull with a cleaned strip..
</div>


Running a business around water incurs increased operational costs. Real estate next to a body of water is more sought after and more expensive, increasing the cost of your rent. Often, a boat will also be required (depending on the application), and boats are not cheap to buy, maintain or park.

The challenges of operating in the space has meant that the robotics revolution is occurring more slowly. The standard for capability, safety and reliability is high due to progress in the land and air domains, yet the state of the art here lags behind, putting larger pressures on businesses to meet expectations. 

The general population is much less aware of marine issues, meaning that takes more effort and education to get people to understand the problem you’re trying to solve, and convince them it’s worth investing in or working on. Take, for example, the issue of hull biofouling, where organisms like slime and barnacles grow quickly on ship hulls. It has three major issues:

- **Emissions:** The shipping industry accounts for over [3% of global emissions](https://safety4sea.com/wp-content/uploads/2021/11/IMO-Biofouling-report-2021_11.pdf), and the fuel burnt just due to drag from these organisms across the whole industry has been estimated as on par with the “[total annual CO2 emissions of a country such as Greece, Nigeria or the Philippines](https://www.glofouling.imo.org/ghg-emissions)”, and quite possibly more since that estimate.
- **Invasive species:** Biofouling also causes the transfer of invasive species between marine environments, and is “[considered to be one of the greatest threats to the world’s freshwater, coastal and marine ecosystems](https://www.glofouling.imo.org/the-issue)”.
- **Hull coatings:** Current hull coatings that reduce biofouling growth are toxic, and probably contribute a [significant quantity of microplastics into the ocean](https://newatlas.com/environment/ship-hull-coatings-microplastic-pollution/).

Those who know that biofouling exists are already a small population, those who know its impacts are a small niche, and those who are trying to solve it (e.g. [Hullbot](https://www.hullbot.com)) are an incredibly small niche for such an important problem. There is also a lack of data and research on many of the issues. With far less exposure, there is less money and therefore less talent and effort going into solving marine issues because of the lack of awareness. And indeed this applies more generally to environmental issues. It makes you think whether the huge amount of effort going into home cleaning robots, could be much better applied to saving the environment.

### Conclusion 
If you've made it this far, I hope you’ve gained some insights into the unique challenges surrounding marine robotics. It is a frontier with huge potential, and a slew of difficult and interesting challenges, technical and otherwise. Whether your talents be in mechanical engineering, design, production, perception, autonomy, business development, raising awareness, or investment, it’s a space where you could have an outsized impact, so I encourage you to dive in!

*Any feedback is welcome! Feel free to shoot me an email using the button at the bottom of the page.*